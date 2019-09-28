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
In old C compilers, variable instantiations must be at he beginning of a function.
In modern C, you can and should create variables at the line, where they are first needed.
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
The old way ist to indicate an error by a return value of -1 and read errno for the error code.
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
