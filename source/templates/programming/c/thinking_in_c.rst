


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


Pointers
===================================
指针是C的重要概念，简单说就是指向一个地址的int值。

const指修饰的变量不能改变，主要看const修饰的位置。例如：

::

    const char* p1; /* *p1='c' illegal; ++p1 OK */
    char* const p2; /* *p2='c' OK; ++p2 illegal */
    const char* const p3; /* no changes at all allowed */

系统即表示    
======================================
unsigned 和 signed转换会给系统带来bug，请看：

::

    size_t strlen(const char*);
    int strlong(char *s, char *t){
        return strlen(s) - strlen(t) > 0;
    }
    /***size_t is defined as unsigned int, it will turn large if it is negative
        correct answer is strlen(s) > strlen(t);
        ************************************************/

    void *memcpy(void *dest, void *src, size_t n);
    #defind KSIZE 1024
    char kbuf[KSIZE];
    int copy_from_kernel(void *user_dest, int maxlen){
        int len = KSIZE < maxlen ? KSIZE : maxlen;
        memcpy(user_dest, kbuf, len);
        return len;
    }
    /***if maxlen is negative it will change large number in memcpy argument, 
        and user will copy many bits of memory.
        FreeBSD source code bug, and it is fixed by
        change maxlen to size_t.
        ****************************************/

