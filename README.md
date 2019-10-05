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
  - [Classes](#S-naming-classes)
  - [Namespaces](#S-naming-namespaces)
  
+ [Object orientation in C](#S-oo)
  - [Simple machine](#S-oo-simple)
  - [Inheritance](#S-oo-inheritance)
  - [Virtual Methods](#S-oo-virtual)
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
    uint8_t data[128];
    bool mode;
};

// the user could instantiate it like so:
struct uncommon uc;
```


#### <a name="S-naming-structs-usecases"></a>Use Cases
As explained in Chapter [Prefer autotypes](#S-basics-autotypes), you should always prefer autotype structs.
Autotype structs should be marked, so the user can directly identify them.
I use (short) lowercasewithoutunderscores names for them:

```c
// Autotype structs

typedef struct {
    float x, y;
} point;

typedef struct {
    point tl, br; // top left, bottom right
} rect;

typedef struct {
    point center;
    float width, height;
    float angle;
} rotatedrect;

typedef struct {
    point data[1024];
    int size;
} pointarray;
```

Structs that own data on heap, or classes, needs to be killed/ freed.
There is a fluid transition between these two, so I treat them the same.
For marking, I use PascalCase for their names.
With this convention, the user directly sees at their instantiation, that he needs to kill them somewhere.

```c
// Classes

typedef struct {
    char *str;
} String;

// Destructor:
void String_kill(String *self) {
    free(self->str);
    self->str = NULL;
}


typedef struct {
    int *data;
    int size;
} IntArray;

// Constructor
void IntArray_init(IntArray *self, int size) {
    self->data = calloc(size, sizeof(int));
    self->size = size;
}

// Destructor
void IntArray_kill(IntArray *self) {
    free(self->data);
    self->data = NULL;
    self->size = NULL;
}

// Method
void IntArray_push(IntArray *self, int append) {
    self->data = realloc(self->data, ++self->size * sizeof(int));
    self->data[self->size-1] = append;
}
```

### <a name="S-naming-classes"></a>Classes
As seen in the previos example above, I prefer PascalCase for classes.
The data section of the class gets the ClassName.
The constructor is called ClassName_init and the destructor ClassName_kill.
All methods also use this naming sheme, like ClassName_length.
With this style and an ide with autocompletion, the user gets a similar feeling to an object orientated language.


### <a name="S-naming-namespaces"></a>Namespaces
If you write a small library with a handful of good used names in the interface, you must not use a namespace.
For example a library that loads a .csv file, can use an interface header like this:

```c 
// No namespace needed

// csv.h
#ifndef CSV_H
#define CSV_H

/**
 * Loads a .csv file into heap memory.
 * @param out_array: Pointer to the allocated data array
 * @param file: The .csv file to load (relative or absolute path, '~' for home)
 * @returns: the number of loaded fields, or -1 on error
 */
int load_csv_file_to_heap_array(float **out_array, const char *file);

/**
 * Saves a .csv file into the given file.
 * @param file: The .csv file to save into (relative or absolute path, '~' for home)
 * @returns: the number of saved fields, or -1 on error
 */
int save_csv_file(const char *file, const float *array, int n);

#endif //CSV_H
```

When your library gets bigger and/ or types get into the interface header, that will be part of interfaces for the user, a namespace is needed.
A namespace is a simple and very short prefix for all names in your interface.
A geometry library may look like the following:

```c
// Namespace (geo)

// geo/types.h
#ifndef GEO_TYPES_H
#define GEO_TYPES_H

typedef float geo_vec[2];

// Autotype struct
typedef struct {
    float x, y;
} geo_point;

typedef struct {
    geo_point center;
    float radius;
} geo_circle;


// Class
typedef struct {
    geo_point *data;
    int size;
} geo_PointArray;

void geo_PointArray_kill(geo_PointArray *self) {
    free(self->data);
    self->data = NULL;
    self->size = 0;
}

#endif // GEO_TYPES_H



// geo/intersection.h
#ifndef GEO_INTERSECTION_H
#define GEO_INTERSECTION_H

#include "types.h"

/**
 * @returns: all points that lie in the circle
 */
geo_PointArray geo_points_in_circle(geo_point *array, int n, geo_circle circle);

#endif // GEO_INTERSECTION_H
```



## <a name="S-oo"></a>Object Orientation in C
Although the C programming language doesn't support object orientated programming nativly,
it's still possible and quite easy.


### <a name="S-oo-simple"></a>Simple machine
A little example of a simple "machine" class was already shown in chapter [Naming structs (use cases)](#S-naming-structs-usecases).
If you know that there will never be more than one instance of your class, go the procedure way:

```c
// max. 1 instance of Foo

//
// foo.h
//

// public data
extern int foo_cnt;

// constructor
void foo_init();

// destructor
void foo_kill();

// methods:
void foo_add(int add);

void foo_print();


//
// foo.c
//

// public data
int foo_cnt;

// private data
static int internal_cnt;

void foo_init() {
    foo_cnt = 1;
    internal_cnt = -1;
}

void foo_kill() {
    // free memory, close files...
    foo_cnt = 0;
}

void foo_add(int add) {
    foo_cnt += add;
}

void foo_print() {
    printf("foo %d\n", foo_cnt+internal_cnt);
}

```

I like to call constructors ClassName_init and destructors ClassName_kill.
If you stick with this, or another name, your users can easily find for other classes as well.
Destructors should always have the function form: void(ClassName *self)
The same "machine", but with multiple possible instances looks like the following:

```c
// multiple instances of foo possible

//
// foo.h
//

typedef struct {
    // public data
    int cnt;
  
    // private data (tailing underscore)
    int internal_cnt_;
} Foo;

// constructor
void Foo_init(Foo *self);

// destructor
void Foo_kill(Foo *self);

// methods:
void Foo_add(Foo *self, int add);

void Foo_print(const Foo *self);

// optional heap constructor
Foo *Foo_new();

// optional heap destructor
void Foo_killfree(Foo *self);


//
// foo.c
//

#include "foo.h" 

void Foo_init(Foo *self) {
    self->cnt = 1;
    self->internal_cnt = -1;
}

void Foo_kill(Foo *self) {
    // free memory, close files...
    self->cnt = 0;
}

void Foo_add(Foo *self, int add) {
    self->cnt += add;
}

void Foo_print(const Foo *self) {
    printf("foo %d\n", self->cnt+self->internal_cnt);
}


Foo *Foo_new() {
    Foo *res = malloc(sizeof(Foo));
    Foo_init(res);
    return res;
}

void Foo_killfree(Foo *self) {
    Foo_kill(self);
    free(self);
}


```


### <a name="S-oo-inheritance"></a>Inheritance
Deriving from a class is easy in C. But its important that your users know the base class.
To derive from the class Foo above, do the following:

```c

typedef struct {
    // include data of mother at first place
    Foo base;

    // public data of Bar
    float amplitude;
} Bar;

void Bar_init(Bar *self, float amp) {
    // call super.init
    // casting to Foo works, 
    //   because the first data in Bar is Foo
    Foo_init((Foo *) self);
    self->amplitude = amp;
}

void Bar_kill(Bar *self) {
    Foo_kill((Foo *) self);
    self->amplitude = -1;
}

void Bar_amplify(Bar *self) {
    self->amplitude *= 2;
}


// Usage:
int main() {
    Bar b;
    Bar_init(&b);
    
    // Call mother method:
    Foo_print((Foo *) &b)
    
    // or...
    Foo_print(&b.base);

    Bar_kill(&b);
}

```


### <a name="S-oo-virtual"></a>Virtual Methods





