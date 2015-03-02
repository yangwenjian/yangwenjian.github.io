


====================================
Thinking In C
====================================


Unary and assignment ops group right-to-left, all others left-to-right.

GCC编译器
====================================
GCC的全称是The GNU Compiler Collection，包含很多语言。

如果编译时包含其他头文件，需要先进行编译其他头文件，才能编译改文件。
例如：

::

    gcc -c mystuff.c
    gcc -c fun.c
    gcc -o fun fun.o mystuff.o

    或者

    gcc -c mystuff.c
    ar -rc mystuff.a mystuff.c
    gcc -c fun.c
    gcc -o fun fun.o mystuff.a

如果是多重引用，需要按照顺序执行。
例如：

::

    gcc -o fun fun.o mystuff.a mystuff2.a
    这里fun调用mystuff中的函数，mystuff调用mystuff2中的函数

如果在c++中引用C的库，需要加入如下代码方可执行：

::

    extern "C"
    {
    #include "mystuff.h"
    }
