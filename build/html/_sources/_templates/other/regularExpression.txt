


================================
Regular Expressions
================================


Introduction
================================
Regular expressions are wildly used in linux&unix programming. With this, we can easily process string and other stuff.

If you want to test something with regular expressions, try this website:

http://www.regexr.com/

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
