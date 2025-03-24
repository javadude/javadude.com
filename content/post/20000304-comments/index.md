---
title: Using the Right Comment in Java
aliases:
- /articles/comments.html

categories:
- Articles

date: "2000-03-04"

description: How and why you should use JavaBean accessor (get/set) methods.

tags:
- java
- language

---

Java provides three types of comments; when should we use which?

<!--more-->

Single-Line (C++-Style) Comments
--------------------------------

The simplest comment in Java is the single line comment. It starts with two forward slashes and continues to the end of the line. For example:
    
```java
// this is a single-line comment
x = 1; // a single-line comment after code
```

Multi-Line (C-Style) Comments
-----------------------------

Java also provides a comment type that can span multiple lines. You start this type of comment with a forward slash followed by an asterisk, and end it with an asterisk followed by a forward slash. The start and end delimiters for this type of comment may be on the same line, or they can be on different lines. For example

```java 
/* This is a c-style comment */

/*  This is also a
    c-style comment, spanning
    multiple lines */
```

Note that C-style comments **_cannot_** be nested. Something like
 
 ```java
/* A comment looks like
    /* This is a comment */
    blah blah blah
*/
```

will generate a syntax error because the java compiler will only treat through the first `*/` as a comment. (The comment ends at the first `*/` the compiler sees.)

You _can_ nest single-line comments within multi-line comments:

```java
/* This is a single-line comment:
    // a single-line comment
*/
```

and multi-line comments in single-line comments:

```java
// /* this is
//    a multi-line
//    comment */
```

Documentation Comments
----------------------
    
Documentation comments are special comments that look like multi-line comments but can be used to generate external documentation about your source code. These begin with a forward slash followed by **_two_** asterisks, and end with an asterisk followed by a forward slash. For example:
    
```java
/** This is a documentation comment */

/** This is also a
    documentation comment */
```

There are a few important things to note about documentation comments:
    
* The documentation generator, javadoc, will add all text inside a documentation comment to an HTML paragraph. This means that any text inside a documentation comment will be formatted into a paragraph; spacing and line breaks are ignored. If you want special formatting, you must include HTML tags inside your documentation comment.*   If a documentation comment begins with more than two asterisks, javadoc assumes this is just used to create a "box" around the comment in the source code and ignores the extra asterisks. For example:

   ```java        
   /**********************************       
       This is the start of a method        
   **********************************/
   ```

   will only keep the text "This is the start of a method".
        
* javadoc will ignore leading asterisks inside a documentation-comment block. For example:

   ```java        
   /***************************************
    * This is a doc comment
    * on multiple lines that I want to stand
    * out in source code, looking "neat"
    ***************************************/
   ```

   will only keep the text "This is a doc comment on multiple lines that I want to stand out in source code, looking "neat""
        
* It's common practice to use something like

   ```java        
   /******************************************
   ...
   ******************************************/
   ```

   to make a comment stand out. Be aware that this is treated as a documentation comment (even if that's not what you had intended), and could show up in the generated documentation.   
    

When to Use Documentation Comments
==================================

Documentation comments should (at very least) be used in front of every public class, interface, method and class/instance variable in your source code. This allows someone to run javadoc against the code and generate a simple document that lists the public entities and a brief description of each. You may also use documentation comments in front on non-public methods, and use a javadoc option to generate documentation for them. Using documentation comments on non-public entities is not as important as publics (the interface isn't exposed...) but if you're commenting the code _anyway_ you might as well write those comments as documentation comments.

When to Use Single-Line Comments
================================

**Always!**

My simple advice on commenting is that whenever you want to write a normal comment (not a documentation comment that describes and class, interface, method or variable) use a single line comment.

Why? Because you can easily use multi-line comments to "comment out" a section of your code! ("Commenting out code" refers to changing the lexical state of a section of source code to being inside a comment, making the compiler ignore that code.) Take as an example:

```java
x = 1;   /* set x to 1 */
y = 2;   /* set y to 2 */
f(x, y); /* call f with x and y */
```

If you want to comment out these three lines, you would either need to put a single line comment in front of each line:

```java
// x = 1;   /* set x to 1 */
// y = 2;   /* set y to 2 */
// f(x, y); /* call f with x and y */
```

or add in multi-line comments wherever there isn't already one present:

```java
/* x = 1;  */ /* set x to 1 */
/* y = 2;  */ /* set y to 2 */
/* f(x, y);*/ /* call f with x and y */
```

or mutilate/remove the "end comment" delimiters of the existing comments:

```java
/*
    x = 1;   /* set x to 1 * /
    y = 2;   /* set y to 2 * /
    f(x, y); /* call f with x and y * /
*/
```

None of these are terribly pleasant. It's much easier if the original code were:

```java
x = 1;   // set x to 1
y = 2;   // set y to 2
f(x, y); // call f with x and y
```

then you can easily comment it out by just placing a multi-line comment around it:

```java
/*
x = 1;   // set x to 1
y = 2;   // set y to 2
f(x, y); // call f with x and y
*/
```

_Always_ use single-line comments for your everyday commenting needs!

NOTE: Most IDEs have a simple "comment-out" function - highlight a block and it comments it out. Most often, it simply prepends `//` at the front of each selected line.

When to Use Multi-Line Comments
===============================

After reading the above section, this becomes obvious. _Only_ use multi-line comments to comment out sections of code. _Never_ use them for _any_ other purpose!
