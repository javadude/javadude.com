---
title: Java is Pass-by-Value, Dammit!
url: /articles/passbyvalue.htm

categories:
- Articles

date: "2001-05-16"

description: Java passes everything _by value_. Period.

tags:
- java
- language

---

I'm really tired of hearing folks incorrectly state "primitives are passed by value, objects are passed by reference".

<!--more-->

I'm a compiler guy at heart. The terms "pass-by-value" semantics and "pass-by-reference" semantics have very precise definitions, and they're often horribly abused when folks talk about Java. I want to correct that... The following is how I'd describe these

## Pass-by-value 

The actual parameter (or argument expression) is fully evaluated and the resulting value is _copied_ into a location being used to hold the formal parameter's value during method/function execution. That location is typically a chunk of memory on the runtime stack for the application (which is how Java handles it), but other languages could choose parameter storage differently.

## Pass-by-reference

The formal parameter merely acts as an _alias_ for the actual parameter. Anytime the method/function uses the formal parameter (for reading or writing), it is actually using the actual parameter.

Java is **_strictly_** pass-by-value, exactly as in C. Read the Java Language Specification (JLS). It's spelled out, and it's correct. In [https://docs.oracle.com/javase/specs/jls/se11/html/jls-8.html#jls-8.4.1](https://docs.oracle.com/javase/specs/jls/se11/html/jls-8.html#jls-8.4.1):

> When the method or constructor is invoked (§15.12), the **_values_** of the actual argument expressions initialize newly created parameter variables, each of the declared type, before execution of the body of the method or constructor. The Identifier that appears in the FormalParameter may be used as a simple name in the body of the method or constructor to refer to the formal parameter.

Note: In the above, _**values**_ is my emphasis, not theirs

Section 15.12.4.2 ([https://docs.oracle.com/javase/specs/jls/se11/html/jls-15.html#jls-15.12.4.2](https://docs.oracle.com/javase/specs/jls/se11/html/jls-15.html#jls-15.12.4.2)) states:

> The argument expressions (possibly rewritten as described above) are now evaluated to yield _argument values_. Each argument value corresponds to exactly one of the method's n formal parameters.

In short: Java **_has_** pointers and is strictly pass-by-value. There are no special rules. It's simple, clean, and clear. (Well, as clear as the evil C++-like syntax will allow ;)

_Note: See the note at the end of this article, "A Note on Remote Method Invocation", for the semantics of remote method invocation (RMI). What is typically called "pass by reference" for remote objects is actually incredibly bad semantics._

* * *

# The Litmus Test

There's a simple "litmus test" for whether a language supports pass-by-reference semantics:

_Can you write a traditional swap(a,b) method/function in the language?_

A traditional swap method or function takes two arguments and swaps them such that variables passed into the function are changed outside the function. Its basic structure looks like

```
swap(Type arg1, Type arg2) {
    Type temp = arg1;
    arg1 = arg2;
    arg2 = temp;
}
```
If you can write such a method/function in your language such that calling

```
Type var1 = ...;
Type var2 = ...;
swap(var1,var2);
```

actually switches the values of the variables `var1` and `var2`, the language supports pass-by-reference semantics.

For example, in Pascal, you can write

```pascal
procedure swap(var arg1, arg2: SomeType);
var
    temp : SomeType;
begin
    temp := arg1;
    arg1 := arg2;
    arg2 := temp;
end;

...

{ in some other procedure/function/program }

var
    var1, var2 : SomeType;

begin
    var1 := ...; { value "A" }
    var2 := ...; { value "B" } 
    swap(var1, var2);
    { now var1 has value "B" and var2 has value "A" }
end;
```

or in C++ you could write

```c++
void swap(SomeType& arg1, Sometype& arg2) {
    SomeType temp = arg1;
    arg1 = arg2;
    arg2 = temp;
}

...

SomeType var1 = ...; // value "A"
SomeType var2 = ...; // value "B"
swap(var1, var2); // swaps their values!
// now var1 has value "B" and var2 has value "A"
```

(Please let me know if my Pascal or C++ has lapsed and I've messed up the syntax...)

But you _cannot_ do this in Java!

* * *

# Now the details

The problem we're facing here is statements like

_In Java, Objects are passed by reference, and primitives are passed by value._

This is half incorrect. Everyone can easily agree that primitives are passed by value; there's no such thing in Java as a pointer/reference to a primitive.

However, _Objects are **not** passed by reference_. A correct statement would be _Object references are passed by value_.

This may seem like splitting hairs, but it is _far_ from it. There is a world of difference in meaning. The following examples should help make the distinction.

In Java, take the case of

```java
public void foo(Dog d) {
    d = new Dog("Fifi"); // creating the "Fifi" dog
}

Dog aDog = new Dog("Max"); // creating the "Max" dog

// at this point, aDog points to the "Max" dog

foo(aDog); 

// aDog still points to the "Max" dog
```

the variable passed in, `aDog`, **_is not_** modified! After calling `foo()`, `aDog` **_still_** points to the `Dog` with name "Max"!

Many people mistakenly think/state that something like

```java
public void foo(Dog d) { 
    d.setName("Fifi");
}
```

shows that Java does in fact pass objects by reference.

The mistake they make is in the definition of

```java
Dog d;
```

itself. When you write that definition, you are defining a _pointer_ to a `Dog` object, _not_ a `Dog` object itself.

## On Pointers versus References...

The problem here is that the folks at Sun made a naming mistake.

In programming language design, a "pointer" is a variable that indirectly tracks the location of some piece of data. The value of a pointer is often the memory address of the data you're interested in. Some languages allow you to manipulate that address; others do not.

A "reference" is an alias to another variable. Any manipulation done to the reference variable directly changes the original variable.

Check out the second sentence of [https://docs.oracle.com/javase/specs/jls/se11/html/jls-4.html#jls-4.3.1](https://docs.oracle.com/javase/specs/jls/se11/html/jls-4.html#jls-4.3.1).

> The reference values (often just references) are pointers to these objects, and a special null reference, which refers to no object.

_They_ explicitly say "pointers" in their description... Interesting...

When they were originally creating Java, they had "pointer" in mind (you can see some remnants of this in classes like `NullPointerException`).

Sun wanted to push Java as a secure language, and one of Java's advantages was that it does not allow pointer _arithmetic_ as C++ does.

They went so far as to try a different name for the concept, formally calling them "references". A _huge_ mistake and it's caused even more confusion in the process.

There's an excellent explanation of reference variables at [http://www.cprogramming.com/tutorial/references.html](http://www.cprogramming.com/tutorial/references.html). It's C++ specific, but it properly says the concept of a true reference variable.

The word "reference" in programming language design originally comes from how you _pass_ data to subroutines/functions/procedures/methods. A reference _parameter_ is an _alias_ to a variable passed as a parameter.

In the end, Sun made a naming mistake that's caused confusion. Java has pointers, and once you accept that, it makes the way Java behaves make much more sense.

## Calling Methods

Calling

```java
foo(d);
```

passes the **_value of `d`_** to `foo()`; it does _not_ pass the object that `d` points to!

The value of the pointer being passed is similar to a memory address. Under the covers it may be a tad different, but from a programmer's perspective, you can think of it in exactly the same way. _The value uniquely identifies some object on the heap_.

**_However,_** it makes _no_ difference how pointers are **_implemented_** under the covers. You program with them **_exactly_** the same way in Java as you would in C or C++. The syntax is just slightly different (another poor choice in Java's design; they should have used the same `->` syntax for de-referencing as C++).

In Java,

```java
Dog d;
```

is **_exactly_** like C++'s

```java
Dog *d;
```

And using

```java
d.setName("Fifi");
```

is exactly like C++'s

```c++
d->setName("Fifi");
```

To sum up: Java **_has_** pointers, and the **_value_** of the **_pointer_** is passed in. There's no way to actually pass an object itself as a parameter. You can only pass a pointer to an object.

Keep in mind, when you call

```java
foo(d);
```

you're not passing an object; you're passing a _pointer_ to the object.

For a slightly different (but still correct) take on this issue, please see Praxis 1 in Peter Haggar's excellent book, _Practical Java_: [https://books.google.com/books?id=iWPeqljHNcoC&lpg=PP1&pg=PA1](https://books.google.com/books?id=iWPeqljHNcoC&lpg=PP1&pg=PA1).

* * *

# A Note on Remote Method Invocation (RMI)

When passing parameters to remote methods, things get a bit more complex. First, we're (usually) dealing with passing data between two independent virtual machines, which might be on separate physical machines as well. Passing the value of a pointer wouldn't do any good, as the target virtual machine doesn't have access to the caller's heap.

You'll often hear "pass by value" and "pass by reference" used with respect to RMI. These terms have more of a "logical" meaning, and really aren't correct for the intended use.

Here's what is usually meant by these phrases with regard to RMI. Note that this is _not_ proper usage of "pass by value" and "pass by reference" semantics:

## RMI "Pass-by-value"

The actual parameter is _serialized_ and passed using a network protocol to the target remote object. Serialization essentially "squeezes" the data out of an object/primitive. On the receiving end, that data is used to build a "clone" of the original object or primitive. Note that this process can be rather expensive if the actual parameters point to large objects (or large graphs of objects).  

**This isn't quite the right use of "pass-by-value"; I think it should really be called something like "pass-by-memento". (See "Design Patterns" by Gamma et al for a description of the Memento pattern).**  
 

## RMI "Pass-by-reference"

The actual parameter, which _is itself a remote object_,  is represented by a proxy. The proxy keeps track of where the actual parameter lives, and anytime the target method uses the formal parameter, _another remote method invocation occurs_ to "call back" to the actual parameter. This can be useful if the actual parameter points to a large object (or graph of objects) and there are few call backs.  

**This isn't quite the right use of "pass-by-reference" (again, you cannot change the actual parameter itself). I think it should be called something like "pass-by-proxy". (Again, see "Design Patterns" for descriptions of the Proxy pattern).**

* * *

# Follow up from stackoverflow.com

_I posted the following as some clarification when a discussion on this article arose on https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value/73021#73021._

The Java Spec says that everything in java is pass-by-value. There is no such thing as "pass-by-reference" in java.

The key to understanding this is that something like

```java
Dog myDog;
```

is not a Dog; it's actually a pointer to a Dog.

What that means, is when you have

```java
Dog myDog = new Dog("Rover");
foo(myDog);
```

you're essentially passing the address of the created Dog object to the foo method. (I say essentially b/c java pointers aren't direct addresses, but it's easiest to think of them that way)

Suppose the Dog object resides at memory address 42. This means we pass 42 to the method.

If the Method were defined as

```java
public void foo(Dog someDog) {  // AAA
    someDog.setName("Max");     // BBB
    someDog = new Dog("Fifi");  // CCC
    someDog.setName("Rowlf");   // DDD
}
```

Let's look at what's happening.

* line AAA

    * the parameter `someDog` is set to the value 42

* line BBB

    * `someDog` is followed to the `Dog` it points to (the `Dog` object at address 42)

    * that `Dog` (the one at address 42) is asked to change his name to "Max"

* line CCC
    * a new `Dog` is created. Let's say he's at address 74.
    *  we assign the parameter `someDog` to that address, 74

* line DDD

    * `someDog` is followed to the `Dog` it points to (the `Dog` object at address 74)
    * that `Dog` (the one at address 74) is asked to change his name to "Rowlf"

Now let's think about what happens outside the method:

_**Did myDog change?**_

There's the key.

Keeping in mind that `myDog` is a pointer, and not an actual `Dog`, the answer is **_NO_**. `myDog` still has the value 42; it's still pointing to the original `Dog`.

It's perfectly valid to follow an address and change what's at the end of it; that does not change the variable, however.

Java works exactly like C. You can assign a pointer, pass the pointer to a method, follow the pointer in the method and change the data that was pointed to. However, you cannot change where that pointer points.

In C++, Ada, Pascal and other languages that support pass-by-reference, you can actually change the variable that was passed.

If Java had pass-by-reference semantics, the foo method we defined above would have changed where `myDog` was pointing when it assigned someDog on line CCC.

Think of reference parameters as being aliases for the variable passed in. When that alias is assigned, so is the variable that was passed in.
