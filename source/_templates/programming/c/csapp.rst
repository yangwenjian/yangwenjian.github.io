


===============================================
CSAPP
===============================================
CSAPP(Computer Science ..)

Capter 1
===============================================

Chapter 2
===============================================
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

