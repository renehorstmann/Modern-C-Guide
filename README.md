# Modern-C-Guide
A Style Guide for modern C programming

In this repository I'll show you my recommendations for good coding style in the C programming language.

IN ACTUAL WORK!

## <a name="S-contents"></a>Contents

+ [Basics](#S-basics)
  - [Where to put variable instantiations](#S-basics-where_variables)
  - [Error handling](#S-basics-errors)
  - [Prefer autotypes](#S-basics-autotypes)

+ Naming
  - Variables
  - Functions
  - Macros
  - Structs
  - Classes
  - Namespaces
  
+ How to use object orientation in C
  - Simple machine
  - Inheritance
  - Virtual Methods
  - Interfaces


## <a name="S-basics"></a>Basics
### <a name="S-basics-where_variables"></a>Where to put variable instantiations
In old C compilers, variable instantiations must be at the beginning of a function.
In modern C, you can and should create variables at the line, where they are first needed.
(If you need a variable multiple times for different use cases (e. g. error codes), put it at the start).
```c
void foo() {
    int sum = 0;

    // declare i within the for loop
    for(int i=0; i<10; i++)
        sum += i*i;

    float inv = 1.0f / sum;

    // bundle complicated assignments
    int *array;
    int size;
    bool use_array;
    {
        if(sum > 20) {
            array = (int*) malloc(16);
            size = 4;
        } else {
            array = (int*) malloc(4);
            size = 1;
        }
        use_array = true;
    }
}
```




### <a name="S-basics-errors"></a>Error Handling
Don't use setjmp and longjmp!
The old way was/ is to indicate an error by a return value of -1 and set a global error code (errno).
A more modern way is to return a static const string, or NULL if no error occured.
If you get an error, you can directly read it in the debug session.
```c
// somewhere in a header
typedef const char *Error;
extern Error ERROR_NullPointer;
extern Error ERROR_FileNotFound;
// ...

// somewhere in a source
Error ERROR_NullPointer = "NullPointer";
Error ERROR_FileNotFound = "FileNotFound";

// function that can throw an error
Error foo(int *a) {
    if(!a)
        return ERROR_NullPointer;

    if(*a < 0)
        return "fooCustomError";

    return NULL; // success
}

// using foo
void bar() {
    Error err = foo(NULL);
    if(err) {

        // direct pointer check
        if(err == ERROR_FileNotFound)
             ; // handle error

        //...
  
        // print the error
        print("Error @ foo: %s\n", err);
    }
}
```

If you have a complicated function, 
in where you must free and close on multiple areas,
break it into multiple smaller functions, 
or use a clean section with gotos, 
or use my [Utilc/Scope](https://github.com/renehorstmann/Utilc).

```c
// bad example:
int complicated() {
    FILE *file = fopen("file.txt", "r");
    int *data;

    // ...

    // pre allocate data
    data = malloc(16);
    if(!data) {
        fclose(file);
        return 1;
    }

    // ...

    // parse file
    float a, b;
    int read = fscanf(file, "%f %f", &a, &b);
    if(read != 2) {
        fclose(file);
        free(data);
        return 2;
    } 

    // ... probably more code with exit conditions

    fclose(file);
    free(data);
    return 0;
}
```
Turn the code above into this:

```c
// splitted function:
static Error parse_file_(const FILE *file, int *data) {
    float a, b;
    int read = fscanf(file, "%f %f", &a, &b);
    if(read != 2)
        return "ParseError";
    
    // ...
    
    return NULL;
}


Error complicated() {
    Error err; // instantiate because of multiple uses

    // pre allocate data
    data = malloc(16);
    if(!data)
        return "AllocError";

    // open file at first usage
    FILE *file = fopen("file.txt", "r");
    err = parse_file_(file, data);
    fclose(file);
    if(err) {
        free(data);
        return err;
    }

    // ...

    free(data);
    return NULL;
}
```

Or/ and use a cleanup section at the functions end:

```c
Error complicated() {
    // In this case, all needed stuff for cleaning
    // should be instantiated here
    Error err = NULL;
    int *data = NULL; // its safe to free(NULL)

    // ...

    // file is only used here, so as above:
    // open file at first usage
    FILE *file = fopen("file.txt", "r");
    err = parse_file_(file, data);
    fclose(file);
    if(err)
        goto CLEAN_UP;
    
    // ...

    CLEAN_UP:
    free(data);
    return err;
}


```


### <a name="S-basics-autotypes"></a>Prefer autotypes
Always prefer autotypes, e. g. use char str[64] instead of char *str_heap = malloc(64).
Its not only faster, but you also dont need to worry about freeing memory.
Structs that represents dynamic arrays can also make use of them:
```c
// An autotype struct that can safe up to 1024 indices (ints)
typedef struct {
    int data[1024];
    int size;
} indices;
```
The disadvantage is of course, that these array autotypes are limited in size, 
but if the contents are small emough, always prefer them.
