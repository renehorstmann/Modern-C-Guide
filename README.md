# Modern-C-Guide
A Style Guide for modern C programming

In this repository I'll show you my recommendations for a good coding style in the C programming language.


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
  - [When not to use](#S-oo-not)
  - [Simple machine](#S-oo-simple)
  - [Inheritance](#S-oo-inheritance)
  - [RTTI](#S-oo-rtti)
  - [Virtual Methods](#S-oo-virtual)
  - [Interfaces](#S-oo-interfaces)


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
If it is possible, always write functions that do not produce any errors at all.
For example:
```c
int count_char(const char *string, char c) {
    if(!string)
        return 0; // simply return 0 if the string is invalid
    int cnt = 0;
    while(*string) {
        if(*string++ == c)
            cnt++;
    }
    return cnt;
}
```

*Todo*

2. Type of error
  - Compile time
  - Debug runtime
  - runtime but should crash everything
  - runtime but should crash the library/module
  - runtime

2. If state would be changed, make the state illegal, so that upcoming functions wont crash

3. How to represent the error
  - global state
    - should get a callback function
  - always returns error value
  - return struct union, created by macro?
  - long jump?

4. What about bindings



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

### <a name="S-oo-not"></a>When not to use
Unlike many do, you should NOT use OOP in every scenario!
In most cases its unaccesary to use all features of it and it can slow down your program.
Imagine you write a game in an OOP manner with the following hierarchy:
+ Item (base class)
  - Invisible
    - Ghost
  - Visible
    - Tree
    - TreasureChest
    - Moveable
      - Player
      - Enemy

So you could have a list of all items (unsorted) and a loop that renders each with an overloaded method render.
This is incredible slow for a normal modern CPU, because of cache misses.
A slightly better approach would be to list all enemies packed in a seperate list and render these.
A much better approuch is to pack all data that is necessary to render an enemy and loop over this list.
In the performance critical section focus on data, not code.


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
If you stick with this, or another name, your users can easily find them for other classes as well.
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
In the following example, the class Child derives (static) from the class Mother:
(static = no run time information and no virtual methods -> the user must call the right methods himself)

```c

// class Mother
typedef struct {
    char *data;
    int a;
} Mother;

void Mother_init(Mother *self, int amount) {
    self->data = malloc(amount);
    self->a = amount;
}

void Mother_kill(Mother *self) {
    free(self->data);
    self->data = NULL;
    self->a = 0;
}

void Mother_print(Mother *self, int pos) {
    assert(pos>=0 && pos<self->a);
    printf("%c", self->data[pos]);
}

// derived class Child
typedef struct {
    // include data of mother at first place
    Mother base;

    // public data of Child
    int b;
} Child;

void Child_init(Child *self, int beta) {
    // call super.init
    // casting to Mother works, 
    //   because the first data in Child is Mother
    Mother_init((Mother *) self, beta*2);
    self->b = beta;
}

void Child_kill(Child *self) {
    Mother_kill((Mother *) self);
    self->b = 0;
}

int Child_length(Child *self) {
    return self->base.a - self->b;
}


// Usage:
int main() {
    Child c;
    Child_init(&c, 10);

    int len = Child_length(&c);
    
    // Call mother method:
    Mother_print((Mother *) &c, 5)
    
    // or...
    Mother_print(&c.base, len);

    Child_kill(&c);
}

```


### <a name="S-oo-rtti"></a>RTTI
Run time type information is needed, to determine the type of a class at runtime (dynamic_cast/ isinstance/ instanceof/ etc.).
To achieve this, the root base class should have an identification string (as char array autotype).
Or all root base classes inherit from an Object class that implements the string.
These strings contain a chained list of the class name hierarchy.
For example if class Bar and class Pub derive from class Foo, their type names would be:
+ class Foo - type "Foo"
  - class Bar : Foo - type "FooBar"
  - class Pub : Foo - type "FooPub"
+ class Car - type "Car"

```c

typedef struct {
    char type[64];
} Object;

void Object_init(Object *self, const char *type) {
    strcpy(self->type, type);
}

void *as_instance(void *object, const char *type) {
    if(strncmp(object, type, strlen(type)) == 0) 
        return object;
    return NULL;
}


// class Foo
typedef struct {
    Object base;

    int a;
} Foo;

const char *Foo_TYPE = "Foo";

void Foo_init(Foo *self) {
    Object_init((Object *)self, Foo_TYPE);
    self->a = 1;
}

// class Bar : Foo
typedef struct {
    Foo base;

    int b;
} Bar;

const char *Bar_TYPE = "FooBar";

void Bar_init(Bar *self) {
    Foo_init((Foo *) self);
    Object_init((Object *) self, Bar_TYPE);
    self->b = 2;
}

// class Pub : Foo
typedef struct {
    Foo base;

    int p;
} Pub;

const char *Pub_TYPE = "FooPub";

void Pun_init(Pub *self) {
    Foo_init((Foo *) self);
    Object_init((Object *) self, Pub_TYPE);
    self->p = 3;
}


// class Car
typedef struct {
    Object base;

    int color;
} Car;

const char *Car_TYPE = "Car";

void Car_init(Car *self) {
    Object_init((Object *) self, Car_TYPE);
    self->color = 0xff00ff;
}


// usage
int main() {
    Bar b;
    Bar_init(&b);

    Foo *as_foo = as_instance(&b, Foo_TYPE);
    if(as_foo)
         puts("Bar is a Foo");
    
    Bar *as_bar = as_instance(as_foo, Bar_TYPE);
    if(as_bar)
         puts("Bar that was casted to Foo still is a Bar");

    Pub *as_pub = as_instance(as_foo, Pub_TYPE);
    assert(!as_pub);

    Car *as_car = as_instance(as_foo, Car_TYPE); 
    assert(!as_car);
}
```

### <a name="S-oo-virtual"></a>Virtual Methods
With virtual methods, the user must not know the exact type, to call the right overlaoded class function (method).
As with the rest of OOP, its easy to implement in C:

```c
// Class Foo
typedef struct Foo {
    // public data
    int f;

    // vtable (function ptr of the virtual methods)::
    void (*print)(const struct Foo *self);
    int (*add)(struct Foo *self, int add);

} Foo;

void Foo_print(const Foo *self) {
    printf("Foo(%d)\n", self->f);
}

int Foo_add(Foo *self, int add) {
    int f = self->f;
    self->f += add;
    return f;
}

void Foo_init(Foo *self) {
    self->f = 1;
    self->print = Foo_print;
    self->add = Foo_add;
}


// Class Bar : Foo
typedef struct {
    Foo base;

    // public data of Bar
    float b;
} Bar;

void Bar_print(const Bar *self) {
    printf("Bar(%d,%f)\n", self->base.f, self->b);
}

int Bar_add(Bar *self, int add) {
    // call super.add
    int foo = Foo_add((Foo *) self, add);

    self->b += (float) foo;
    return foo;
}

void Bar_init(Bar *self, float init) {
    // call super.init
    Foo_init((Foo *) self);

    self->b = init;

    // change overloaded vtable methods
    self->base.print = (void(*)(const Foo *)) Bar_print;  // optional cast...
    self->base.add = (int(*)(Foo *, int)) Bar_add;
}


// Usage
int main() {
    Foo foo;
    Foo_init(&foo);

    Bar bar;
    Bar_init(&bar, 1.23f);


    Foo *foos[2] = {&foo, (Foo *) &bar}; // optional cast...

    for(int i=0; i<2; i++) {
        Foo *f = foos[i];
        f->add(f, 10);
        f->print(f);
    }

}

```




### <a name="S-oo-interfaces"></a>Interfaces
Often interfaces perform a better job, compared to inheritance, to provide an easy OOP-feel.
An interface only consists of virtual methods and so is like an abstract class without data.
In C you also must also add an void * for the implementation:

```c

typedef struct Printable {
    void *impl_;

    void (*print)(struct Printable self);
} Printable;


// class Foo
typedef struct {
    float f;

    Printable printable;
} Foo;

// function that takes a Printable
void bar(Printable p, int n) {
    for(int i=0; i<n; i++) 
        p.print(p);
}



void Foo_print(Printable self) {
    Foo *foo = (Foo *) self.impl_;
    printf("Foo(%f)\n", foo->f);
}

void Foo_init(Foo *self) {
    self->f = 0;
    self->printable = (Printable) {
        (void *) self,    // opt. cast
        Foo_print
    };
}


// usage
int main() {
    Foo foo;
    Foo_init(&foo);
    foo.f = 1.23f;
    
    bar(foo.printable, 3);
}

```
