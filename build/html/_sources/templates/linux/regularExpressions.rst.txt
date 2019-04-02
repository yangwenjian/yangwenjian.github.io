


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
这里记录一些自己写过的表达式，供参考：

数字的正则表达式：

::

    正整数和小数：/^([0-9]*[1-9][0-9]*\.[0-9]+|0\.[0-9]*[1-9][0-9]*|([0-9]*[1-9][0-9]*))$/g


IP地址的正则表达式：

::

    通用的IP表达式：/\b(([01]?\d?\d|2[0-4]\d|25[0-5])\.){3}([01]?\d?\d|2[0-4]\d|25[0-5])\b/g
    精确的IP表达式：/\b(1\d?\d?|[1-9]\d|[1-9]|2[0-4]\d|25[0-5])\.((1\d?\d?|[1-9]\d|\d|2[0-4]\d|25[0-5])\.){2}(1\d?\d?|[1-9]\d|[1-9]|2[0-4]\d|25[0-5])\b/g

Regular expression in find and grep
-------------------------------------------
在linux中可以使用正则表达式进行搜索文件：

::

    通用表达式：find PATH -regex '.*REGX'(必须以.*为开头，否则不能匹配上)

在搜索文件内容时也可使用：

::

    通用表达式：find PATH -type f -name '*' | xargs grep 'REGX'(这里需要加入-type选项，要不然所有文件夹都会被列出)
    

这里也从网上找一些典型的正则表达式：

::

    匹配HTML标记：<\s*(\S+)(\s[^>]*)?>[\s\S]*<\s*\/\1\s*>
    
参考资料
-----------------------------------
http://www.regexr.com/
