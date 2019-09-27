# C-Style-Guide
A Style Guide for modern C programming

In this repository I'll show you my recommendations for good coding style in the C programming language.

IN ACTUAL WORK!

## <a name="S-contents"></a>Contents

+ [Basics](#S-basics)
  - [Where to put variable instantiations](#S-basics-where_variables)
  - Error handling
  - Prefer autotypes

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
In modern C, you would create variables at the line, where they are first needed.
```c
void foo() {
    int sum = 0;

    // declare i within the for
    for(int i=0; i<10; i++)
        sum += i*i;

    float inv = 1.0f / sum;

    // bundle complicated assignments before
    int *array;
    int size;
    {
        if(sum > 20) {
            array = (int) malloc(16);
            size = 4;
        } else {
            array = (int) malloc(4);
            size = 1;
        }
    }
}
```
