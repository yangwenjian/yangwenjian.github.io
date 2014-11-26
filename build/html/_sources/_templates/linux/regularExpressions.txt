


================================
Regular Expressions
================================


Introduction
================================
Regular expressions are wildly used in linux&unix programming. With this, we can easily process string and other stuff.

同样的匹配可以用不同的正则表达式实现，但是需要注意匹配的效率。

Usage
--------------------------------
/[A-Z]+(abc|xyz)*/i

first, character '/' means a start;
[A-Z] means any character between letter A and Z;
a '+' means match one or more expression before this charactor;
(abc|xyz) means specific mathes abc or xyz;
* means match zero or more expression before this charactor;
character '/' means an end;
last, the letter after end character means option of this regular expression, in this case, 'i' means case insensitive.

Exercise
---------------------------------
IP地址的正则表达式：

::

    /\b(([01]?\d?\d|2[0-4]\d|25[0-5])\.){3}([01]?\d?\d|2[0-4]\d|25[0-5])\b/g

也有其他的表达方式，这里只列举一种，\b表示开始和结束，?表示0或者1个。如果要剔除0开头和结尾的IP，写得就复杂一些：

::

    /\b(1\d?\d?|[1-9]\d|[1-9]|2[0-4]\d|25[0-5])\.((1\d?\d?|[1-9]\d|\d|2[0-4]\d|25[0-5])\.){2}(1\d?\d?|[1-9]\d|[1-9]|2[0-4]\d|25[0-5])\b/g

这里从网上找一些典型的正则表达式：

::

    匹配HTML标记：<\s*(\S+)(\s[^>]*)?>[\s\S]*<\s*\/\1\s*>
    
参考资料
-----------------------------------
http://www.regexr.com/
