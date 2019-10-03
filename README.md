# Modern-C-Guide
A Style Guide for modern C programming

In this repository I'll show you my recommendations for a good coding style in the C programming language.

IN ACTIVE WORK!

## <a name="S-contents"></a>Contents

+ [Basics](#S-basics)
  - [Where to put variable instantiations](#S-basics-where_variables)
  - [Error handling](#S-basics-errors)
  - [Prefer autotypes](#S-basics-autotypes)

+ [Naming](#S-naming)
  - [Variables](#S-naming-variables)
  - [Functions](#S-naming-functions)
  - [Macros](#S-naming-macros)
  - [Structs](#S-naming-structs)
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
break it into small helper functions, 
or use a clean section with gotos, 
or use my [Utilc/Scope](https://github.com/renehorstmann/Utilc#S-Scope).

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



## <a name="S-naming"></a>Naming
A consistent naming sheme is generelly usefull for all kinds of stuff, but especially in programming and its very very important in C programming.
In the C programming world, there is no specific definition of, for example, a class and its methods.
Its the programmers task to use a naming sheme, so that the reader can easily see whats going on.
With autocompletion, there is no need for small names like strtof. The programmer should always write readable code, with only common and/ or good abbreviations.


### <a name="S-naming-variables"></a>Variables
I prefer to use snake_case names for variables, if you want to use a member of a struct and you have no clue, use its lowercase name or a good abbreviation.
```c
int car_petrol;
FILE *file;
strviu viu;
intiterator iter;
```

### <a name="S-naming-functions"></a>Functions
Like the variables [above](#S-naming-variables), I also prefer snake_case names.
```c
FILE *open_and_check(const char *filepath);
intset set_diff(intset a, intset b);
int max(int a, int b);
```

### <a name="S-naming-macros"></a>Macros
Lots of C programmers use SCREAM_CASE for macros. But it leads to errors if these are reset by other libraries.
If you want to use SCREAM_CASE, always use a namespace prefix like MYLIB_SCREAM_CASE (MYLIB should be replaced...).
Instead of using this, I prefer PascalCase for macros:
```c
#define Max(a, b) ((a) > (b) ? (a) : (b))
#define Free0(ptr) {free(ptr); ptr=NULL;}
```


### <a name="S-naming-structs"></a>Structs
Structs can occour in three [code areas](#S-naming-structs-area):
+ Implementation
+ Interface header with uncommon use
+ Interface header with common use

With one of the following three [use cases](#S-naming-structs-usecases):
+ Autotype Structs
+ Structs that needs to be freed/ killed
+ Classes

#### <a name="S-naming-structs-area"></a>Code Areas
Within an implementation, or when commonly used in an interface header,
create the struct with a typedef:

```c
// Implementation & Interface header with common use

typedef struct {
    int a, b, c;
} foo;

// or
typedef struct item {
    int i;
    struct item *next;
} item;
```

If its a not commonly used struct in an interface header, 
don't use a typedef. The user can than self decide if he want to create it.
In this way, the name of the struct is not wasted for the user (except for structs).

```c
// Interface header with uncommon use

struct uncommon {
    bool mode;
    uint8_t data[128];
};

// the user could instantiate it like so:
struct uncommon uc;
```


#### <a name="S-naming-structs-usecases"></a>Use Cases
As explained in Chapter [Prefer autotypes](#S-basics-autotypes), you should always prefer autotype structs.

