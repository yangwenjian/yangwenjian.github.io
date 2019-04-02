


========================================
Fork Function
========================================

Best Exercise
========================================
See code here:

.. code:: cpp
    
    #include <stdio.h>
    #include <sys/types.h>
    #include <unistd.h>
    int main(void){
	    int i;
	    for(i = 0; i < 2; i++){
		    fork();
		    printf("hello");
	    }
	    return 0;
    }

It will print 8 times.

But if we change the print content to "hello\n", the result will be 6 times.

Why?

I compare the assembly code of these two, and I find one simple difference.

.. code:: 

    //printf("hello\n")
    .L3:
        call    fork
        movl    $.LC0, %edi
        movl    $0, %eax
        call    printf
        addl    $1, -4(%rbp)

    //printf("hello")
    .L3:
        call    fork
        movl    $.LC0, %edi
        call    puts
        addl    $1, -4(%rbp)

Puts function will print the string immidiately; while printf will put the string into buffer area.
