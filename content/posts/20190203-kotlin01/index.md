---
title: From Java to Kotlin - Episode 1 - You've Gotta Start Somewhere...

categories:
- Articles

date: "2019-02-03"

tags:
- kotlin
- language

---

<script src="https://unpkg.com/kotlin-playground@1" data-selector="pre"></script>

<table>
	<tr>
		<td style="border:none;background: unset;"><a href="https://www.jetbrains.com"><img src="../../img/kotlin-logo.png" alt="Kotlin Logo" style="width:300px;"/></a></td>
		<td style="border:none;background: unset;"><font size="+2"><p>Let's learn Kotlin from a Java point of view.<p>We'll start by looking at some basics of Variables, Values, Classes, Properties and Constructors.</p></font></td>
	</tr>
</table>

<!--more-->

Introduction
--------------------------------
I remember the way I felt when Java came out. There was this feeling of freedom using Java after using C++ for so long. I feel the same way switching from Java to Kotlin...

This series of posts assumes you're fairly familiar with Java. I'll explain some Java concepts more than others (like Anonymous Inner Classes and Lambdas), and in some cases point you to other articles or videos for more detail on how Java does things. 

Hopefully, many of the Kotlin concepts will make sense on their own... But don't say I didn't warn you ;)

This post starts by taking a look at the very basics of Variables, Values, Classes, Properties and Constructors. In future articles, I'll be going into more detail on these topics. Learning a new programming language often requires a lot of back and forth as the concepts are often highly interconnected...

A Note on Nulls
-------------------------------
While you're reading this, you'll likely think "why didn't he use null instead of dummy values?"...

Nullability is a pretty big concept and I knew this article would be pretty long to start with, so I'm punting it down the road a wee bit. But don't worry... I'll get to it... It's quite important!


Examples
-------------------------------
I'm using the terribly-cool Kotlin Playground that JetBrains has set up. Every example in these posts can be run by clicking the little green arrow, _and_ you can even modify the code if you'd like. (If you modify the code and want to reset it, you can refresh the page and the original code will be restored.)

Some of the longer examples might initially hide code that you've already seen. In these cases, you'll see a little "+" icon at the top of the example. Clicking the "+" will expose the rest of the example. Note that the hidden code will be read-only and have a darker background.

(Note that the Java code that appears in these articles cannot be executed in the Kotlin Playground.)

For more details on Kotlin Playground, see https://github.com/JetBrains/kotlin-playground

Note that if you do not have JavaScript enabled, the code examples will appear as normal text blocks.

Main Functions
--------------------------------
Kotlin has "main" functions just like Java. We'll use this to run examples in most of these articles. You'll most likely use Kotlin in environments like servers or Android applications that don't need a main, but they're useful for examples.

If you don't need any command-line arguments:
<pre theme="darcula" lines="true" match-brackets="true">
fun main() {
	println("Hello, World!")
}
</pre>

If you want command-line arguments, you can use an explicit Array (more on that and `joinToString()` later). I'm passing `A B C` to Kotlin Playground for this example.
<pre args="A B C" theme="darcula" lines="true" match-brackets="true">
fun main(args : Array&lt;String>) {
	println("Hello, World!")
	println(args.joinToString())
}
</pre>

or you can use a varying-length argument list (more on that and `joinToString()` later). I'm passing `A B C` to Kotlin Playground for this example.
<pre args="A B C" theme="darcula" lines="true" match-brackets="true">
fun main(vararg args : String) {
	println("Hello, World!")
	println(args.joinToString())
}
</pre>


Yay! No Semicolons!
--------------------------------
Ding, dong, semicolons are dead! Well, not dead, just not as required as in Java. You can still use them to separate statements, but I don't recommend it. Give yourself a little time to get used to it and you'll add 5 more years before you get carpal-tunnel syndrome. At least according to the science I made up in my head.

(I do find myself forgetting semicolons when I have to write Java code... But I don't do that often any more...)


Variables and Values
--------------------------------
In Kotlin, we explicitly state whether a local or member of a class can be modified. We do this by specifying `var` or `val` to define a variable or value.

Try running the following example (press the green arrow) and you'll see the error.

<pre theme="darcula" lines="true" match-brackets="true">
fun main() {
	var x : String = "Hello"
	val y : String = "Message"

	x = "aaa" // just fine; can change the variable
	y = "bbb" // will not compile!!! can't change a "val"

	println(x)
	println(y)
}
</pre>

When you run the above example, note the "initializer is redundant" warning for line 2. Kotlin does a lot of data flow analysis, and here it's telling us that it sees the only path using `x` assigns it twice before using it, so the initializer isn't needed. We could write

<pre theme="darcula" lines="true" match-brackets="true">
fun main() {
	var x : String
	val y : String = "Message"

	x = "aaa" // just fine; can change the variable

	println(x)
	println(y)
}
</pre>

Type Inferencing
--------------------------------
But things get even better. Kotlin loves to _infer_ types so you don't have to write as much code. For example:

<pre theme="darcula" lines="true" match-brackets="true">
fun main() {
	val x = "aaa"
	println(x)
}
</pre>

The expression `"aaa"` is of type `String`, so kotlin knows that the value `x` can be a `String` and infers the type.

Everything is Public and Final
--------------------------------
First, the good news...

All types, functions and properties are `public` by default. I think this was a great choice, as I'm a big proponent of keeping things accessible unless there's a really strong reason not to (that reason is usually a sensitive algorithm, such as a security feature). `public` here means the same thing as in Java; any class can access `public` types, functions and properties.

However... 

All classes, functions and properties are `final` by default (unless explicitly marked as `open`, `abstract` or `sealed`), or being functions/properties defined inside an `interface`). Interfaces are always non-final (as a `final` interface wouldn't make sense...)

**[Here comes the rant!!!]**

This was a _horrible_ design choice, IMNSHO. This means you cannot create subclasses, or override functions or properties unless the parent explicitly says so. Unfortunately the designers of the language took advice from "Effective Java" (which says to only make non-final if you intend people to extend) a little too far. I've come across far too many cases where a library doesn't let you extend something that has gotten in the way. I believe in the Open/Closed principal - open for extension, closed for modification of its guts.

**[rant over... you may continue with your lives...]**


Class and Interface Inheritance
--------------------------------
Classes/Interfaces (and their members) in Kotlin are nowhere near as verbose as in Java. 

<pre theme="darcula" lines="true" match-brackets="true">
interface Foo
open class A
open class B : A(), Foo
class C : B()

fun main() {
	val c = C()
}
</pre>

Let's break this down, line by line
   
Line | What's Happening
-----|-------------------
1    | Defines an interface named `Foo`. Nothing in it, but we might just be using it as a Marker to help us classify objects.
2    | Defines a class named `A`<br/>- We can extend it because it's marked `open`
3    | Defines a class named `B`<br/>- It is a subclass of class `A`<br/>- It implements interface `Foo`<br/>- It can be extended because it's `open`
4    | Defines a class named `C`<br/>- It is a subclass of `B`<br/>- Note that this means it's indirectly a subclass of `A` and implements `Foo`<br/>- It _cannot_ be extended because it is _not_ explicitly marked `open`
7    | Create an instance of `C` and point value `c` to it. (Yes... Kotlin has pointers, just like Java... See [Java is Pass-By-Value, Dammit!](../../articles/passbyvalue.htm))

Some interesting things to note
   
   - Both interfaces and a class can appear after the colon.
      - there can be at most one class, and any number of interfaces, in any order
   - Superclasses must be followed by a call to a constructor (the parentheses in `A()` and `B()` on lines 3 and 4)

Before we talk about constructors, let's talk about properties, and I'll circle back. I promise...

Properties
------------------------
I've been waiting for these for a long time... There's a concept used in Java called JavaBeans, part of which is a convention for defining properties. I've got an [article on JavaBeans](../20000304-javabeans) that goes into a ton of detail, but here are the basics.

### Properties in Java
A JavaBean property is defined by one or two methods in a Java class:

<pre mode="java" theme="darcula" lines="true" match-brackets="true">
// JAVA CODE
public String getFirstName() { ... }
public void setFirstName(String firstName) { ... }
</pre>

   - If _only_ a `get` method exists, we're defining a read-only property. For example, if we only had `getFirstName()` we're defining a read-only property called `firstName` (note the case.)
   - If _only_ a `set` method exists, we're defining a write-only property. For example, if we only had `setFirstName()` we're defining a write-only property called `firstName`
   - If _both_ methods exist, we're defining a read/write property

These conventions could be used by various tools, such as GUI builders, to automatically determine properties that a user could configure when designing an application. These properties would often appear in a little table where the user could set values and see how the GUI changes, then generate code to set that value at runtime.

But this is horribly verbose, and nearly all `get`/`set` methods look identical in an application.

### Properties in Kotlin

Kotlin fixes this by introducing properties as first-class language constructs.

<pre theme="darcula" lines="true" match-brackets="true">
class A {
	var firstName : String = "no first name"
	var lastName = "no last name"
}

fun main() {
	val a = A()
	println(a.firstName)	
	a.firstName = "Scott"
	println(a.firstName)	
}
</pre>

Adding `var` and `val` definitions inside a class or interface defines a property. `var` properties are read/write (you can modify them), and `val` properties are read-only (you cannot modify them).

Let's look at this example line-by-line

Line | What's Happening
-----|-------------------
1    | Define a new class (not extensible!)
2    |  Define a read/write property called `firstName` of type `String` initialized to `"no first name"`
3    | Define a read/write property called `lastName` of _inferred_ type `String` initialized to `"no last name"`
7    | Create an instance of class `A` and point value `a` to it. Note that there is no `new` keyword; we just specify the class name followed by parens and constructor parameters (if any)
8    | Follow pointer `a` to its `A` instance and print the value of its `firstName` property
9    | Follow pointer `a` to its `A` instance and change the value of its `firstName` property to `"Scott"` <br/>(Note that technically we're setting the value of the property to _point_ to a `String` with the value `"Scott"`)
10   | Follow pointer `a` to its `A` instance and print the value of its `firstName` property

### Using Kotlin Properties From Java Code
Kotlin has excellent interoperaility with Java. If we defined a Kotlin class

<pre theme="darcula" lines="true" match-brackets="true" data-highlight-only>
class A {
	var firstName : String = "no first name"
	var lastName = "no last name"
}
</pre>

we could access it from Java as

<pre mode="java" theme="darcula" lines="true" match-brackets="true">
// JAVA CODE
A a = new A();
a.setFirstName("Scott"));
System.out.println(a.getFirstName());
</pre>

Behind the scenes, Kotlin creates a class file to run in the Java Virtual Machine, and this class file contains `get` and `set` methods for each read/write property, or just `get` methods for read-only properties.

### Backing Fields
To explain "Backing fields" in Kotlin, let's look at a typical property implementation in Java

Remember that JavaBean properties are defined _solely on the presence of `get` and `set` methods_. The implementation of those methods does not matter. However, we typically need to store a value when `set` is called and return it when `get` is called. For example:

<pre mode="java" theme="darcula" lines="true" match-brackets="true">
// JAVA CODE
class Foo {
	private String name;
	public String getName() {
		return this.name;
	}
	public void setName(String name) {
		this.name = name;
	}
}
</pre>

Note the field `name` in this example. It's a field in the class that's hidden from anything outside of class Foo. The property defined by `getName` and `setName` uses the `name` field to store the property value.

Kotlin properties can automatically create a behind-the-scenes field just like this. For example, when you write

<pre theme="darcula" lines="true" match-brackets="true" data-highlight-only>
class Foo {
	var name : String = "no name"
}
</pre>

the Kotlin compiler generates a `get` and `set` function for us (note that Kotlin uses the name "function" for what we would call a "method" in Java), and a "backing field" to hold the value of the `name` property.

But that doesn't _always_ happen...

In Java, we might define a `get` method that computes a value (often called a "derived property") or returns a literal value. For example

<pre mode="java" theme="darcula" lines="true" match-brackets="true">
// JAVA CODE
class Foo {
	public String getName() {
		return "Scott";
	}
}
</pre>

In this case, we don't need a field to store the value.

If we do something like the following in Kotlin

<pre theme="darcula" lines="true" match-brackets="true" data-highlight-only>
class Foo {
	val name : String = "no name"
	   // NOTE: a "val" property, so only defines a getter!
}
</pre>

It _initializes_ the backing field to "no name" and returns it whenever the property is accessed.

If we want the equivalent of that literal-value `get` method in Java, we need to explicitly define what the `get` function in Kotlin would look like. We do this as follows:

<pre theme="darcula" lines="true" match-brackets="true" data-highlight-only>
class Foo {
	val name : String
		get() {
			return "no name"
		}
}
</pre>

In this case, because we explicitly define the get() function, and do not mention the backing field, no backing field is defined.

So how do we explicitly mention the backing field? We use the `field` keyword. For example:

<pre theme="darcula" lines="true" match-brackets="true" data-highlight-only>
class Foo {
	val name : String = "no name"
		get() {
			return field
		}
}
</pre>

This example is pretty silly; we're explicitly defining the default behavior of a read-only property. It's more interesting if we want to modify the behavior. For example, we could print or log a message whenever the field is requested

<pre theme="darcula" lines="true" match-brackets="true" data-highlight-only>
class Foo {
	val name : String = "no name"
		get() {
			println("name requested!")
			return field
		}
}
</pre>

Or more usefully, do something when a value is set. For example, suppose we wanted to ensure a Doctor was always called "Dr.":

<pre theme="darcula" lines="true" match-brackets="true">
class Doctor {
    var name : String = "Dr. Nobody"
        set(value) {
            if (!value.startsWith("Dr. ")) {
                field = "Dr. " + value
            } else {
                field = value
            }
        }
}

fun main() {
    val doc = Doctor()
    doc.name = "Scott"
    println(doc.name)
}
</pre>

(NOTE: There are some much more "Kotlin-y" ways to write the body of that `set` function, but I wanted to keep it closer to Java until we get to those concepts and idioms)

Here we're explictly defining a `set` function for the `name` property. We look at the value passed in (the type of `value` is inferred to be `String`, the same as the type of the property), and if it doesn't call me a "Dr." (which I am not; you can just call me "Scott") it prepends "Dr."" when setting the value in the backing field.

We're not defining the `get` function so we get its default behavior.

Most of the time, you'll use properties without explicitly defining the `get` or `set` functions, but occasionally these are useful.

The most-common instance of defining a `get` is for returning literals. This can be a little more efficient than initializing the property value and having to keep an extra backing field inside the class.

Constructors
------------------------------
NOTE: There is a _lot_ of code in this section that can be significantly reduced; I'm starting the section conceptually similar to how you would write the equivalent Java code, followed by a section on simplifying it after we've finished the basic concepts.

Now that we've defined the basics of properties (there will be much more cool stuff to come later...), we can talk about how Constructors work.

Kotlin wants to be terse, which is good, because I don't like to type. (He says after typing in everything you've just read...) One of it's best tricks is to combine the definition of a constructor directly in the definition of the class. 

Let's take a look at how we can pass in a name to a `Person` instance when creating it.

<pre theme="darcula" lines="true" match-brackets="true">
class Person(name : String) {
    var name : String = "No Name"
    init {
        this.name = name
    }
}

fun main() {
    val person = Person("Scott")
    println(person.name)
}
</pre>

Let's look at this example line-by-line

Line | What's Happening
-----|-------------------
1    | Define a new class named `Person` with a _primary constructor_.<br/>This primary constructor takes a `name` parameter, which is `final` (in Java terms) and is of type `String`.
2    | Define a read/write property called `name` of type `String` initialized to `"No name"`
3    | Define an initializer. This is a block of code that will run as the body of the primary constructor.
4    | Assign the property `name` to the constructor parameter `name`. The `this.` qualifies it to distinguish the property from the parameter. Alternatively, we could have just used a unique name for the parameter and not needed the `this.` qualification.

Kotlin separates the concept of primary and secondary constructors. The _Primary Constructor_ is specified directly after the name of the class. _Secondary Constructors_ are specified in the body of the class.

### Secondary Constructors

You can define alternative constructors. For example, if we wanted to be able to skip passing in a value for the name we could define the following

<pre theme="darcula" lines="true" match-brackets="true">
class Person(name : String) {
    var name : String = "No Name"
    constructor() : this("No Name")
    init {
        this.name = name
    }
}

fun main() {
    val person = Person("Scott")
    println(person.name)
    val person2 = Person()
    println(person2.name)
}
</pre>

On line 3 we're defining a _secondary constructor_ that calls the primary constructor passing in "No Name". You can have any number of secondary constructors.

If you have defined a _primary constructor_, **all** _secondary constructors_ must directly or indirectly call it by using the `constructor(...) : this(...)` syntax. For example:

<pre theme="darcula" lines="true" match-brackets="true">
class Person(name : String) {
    var name : String = "No Name"
    constructor(n : Int) : this("No Name " + n)
    constructor() : this(42)
    init {
        this.name = name
    }
}

fun main() {
    val person = Person("Scott")
    println(person.name)
    val person2 = Person()
    println(person2.name)
    val person3 = Person(10)
    println(person3.name)
}
</pre>

This time, line 3 defines a secondary that takes an `Int` (similar to Java's primitive `int` behind the scenes, but treated like an object in code) and appends it after "No Name" before passing it to the primary constructor.

Line 4 defines another secondary constructor that takes no parameters, and passes `42` to the other secondary constructor, which will then append it to "No Name" and pass it to the primary constructor.

The `init` block then runs as the body of the primary constructor to set the `name` property.

### Calling Superclass Constructors
If you only need to call the primary constructor from a subclass, things are pretty simple:

<pre theme="darcula" lines="true" match-brackets="true">
open class Person(name : String) {
    var name : String = "No Name"
    init {
        this.name = name
    }
}

class Student(name : String) : Person(name)

fun main() {
    val student = Student("Scott")
    println(student.name)
}
</pre>

Note that `Person` is defined as `open` so we can create subclasses, and has a primary constructor. We call that primary constructor on line 8, right after the superclass name, passing in the value passed to the primary constructor of Student. Student can do more than that, of course. For example:

<pre theme="darcula" lines="true" match-brackets="true">
open class Person(name : String) {
    var name : String = "No Name"
    init {
        this.name = name
    }
}

class Student(name : String, gpa : Float) : Person(name) {
	var gpa : Float = 0F
	init {
		this.gpa = gpa
	}
}

fun main() {
    val student = Student("Scott", 3.99F) // Sooooo close...
    println(student.name)
    println(student.gpa)
}
</pre>

Things get a little trickier if you want to call secondary constructors in a superclass. In that case, _you cannot define a primary constructor in the subclass_. If you define a primary constructor, _all_ secondary constructors must call it, which doesn't give us the choice of which super constructor to call... Here's an example:

<pre theme="darcula" lines="true" match-brackets="true">
open class Person(name : String) {
    constructor() : this("No Name")
    var name : String = "No Name"
    init {
        this.name = name
    }
}

class Student : Person {
    constructor() : super()
    constructor(name : String) : super(name)
    var gpa : Float = 0F
}

fun main() {
    val student = Student("Scott")
    student.gpa = 3.99F
    val student2 = Student()
    student2.gpa = 3.50F
    println(student.name)
    println(student.gpa)
    println(student2.name)
    println(student2.gpa)
}
</pre>

So we can pick and choose which superclass constructors are called, but this makes it impossible to pass the gpa to a student constructor and assign it! Time to start looking at better ways to write the code we've been seeing...

Simplifying Constructors
------------------------------
Let's introduce some new concepts that can make constuctors much simpler...

### Default Parameter Values
First, default values for parameters...

Most of the time, we define alternate constructors just to provide default values or different subsets of parameters. Let's start by defining a primary constructor for Person that takes the `name` and gives it a default value.

<pre theme="darcula" lines="true" match-brackets="true">
open class Person(name : String = "No Name") {
    var name : String = "No Name"
    init {
        this.name = name
    }
}

fun main() {
    val person1 = Person()
    val person2 = Person("Scott")
    println(person1.name)
    println(person2.name)
}
</pre>

Now we can create a `Person` with or without a name using the same constructor. Let's add the `Student` subclass:

<pre theme="darcula" lines="true" match-brackets="true">
open class Person(name : String = "No Name") {
    var name : String = "No Name"
    init {
        this.name = name
    }
}

class Student(gpa : Float, name : String = "No Name") : Person(name) {
    var gpa : Float = 0F
    init {
        this.gpa = gpa
    }
}

fun main() {
    val person1 = Person()
    val person2 = Person("Scott")
    println(person1.name)
    println(person2.name)

    val student = Student(3.99F, "Scott")
    val student2 = Student(3.5F)
    println(student.name)
    println(student.gpa)
    println(student2.name)
    println(student2.gpa)
}
</pre>

Now we're able to require the GPA in the `Student` constructor! Note that parameters with default values must appear _after_ any parameters that _do not_ have default values... Unless...

### Naming Parameters in a Call
Kotlin allows you to explicitly name your parameters when calling a function or constructor. For example:

<pre theme="darcula" lines="true" match-brackets="true">
open class Person(name : String = "No Name") {
    var name : String = "No Name"
    init {
        this.name = name
    }
}

class Student(gpa : Float, name : String = "No Name") : Person(name) {
    var gpa : Float = 0F
    init {
        this.gpa = gpa
    }
}

//sampleStart
fun main() {
    val person1 = Person()
    val person2 = Person(name="Scott")
    println(person1.name)
    println(person2.name)

    val student = Student(3.99F, name="Scott")
    val student2 = Student(3.5F)
    println(student.name)
    println(student.gpa)
    println(student2.name)
    println(student2.gpa)
}
//sampleEnd
</pre>

This also allows us to specify parameters out of order, or even define default-valued parameters before non-default-valued parameters:

<pre theme="darcula" lines="true" match-brackets="true">
open class Person(name : String = "No Name") {
    var name : String = "No Name"
    init {
        this.name = name
    }
}

// NOTE PARAMETER ORDER CHANGE!!!
class Student(name : String = "No Name", gpa : Float) : Person(name) { 
    var gpa : Float = 0F
    init {
        this.gpa = gpa
    }
}

fun main() {
    val person1 = Person()
    val person2 = Person("Scott")
    println(person1.name)
    println(person2.name)

    val student = Student("Scott", 3.99F) // ordered
    val student2 = Student(gpa=3.5F) // named
    val student3 = Student(gpa=3.99F, name="Scott") // different order!
    println(student.name)
    println(student.gpa)
    println(student2.name)
    println(student2.gpa)
    println(student3.name)
    println(student3.gpa)
}
</pre>

Let's look at a few lines in particular

Line | What's Happening
-----|-------------------
9    | we swapped the order of the parameters; the `name` with a default value, comes first. (NOTE: We can no longer call the constructor without a name specified _unless_ we name the `gpa` parameter in the call!)
22   | We create a `Student` instance with positional parameters as before, but the parameter order is reversed in the constructor definition (line 9).
23   | We create a `Student` instance _without_ a name passed in. Note that we _must_ name the gpa parameter, as it follows a default-valued parameter in the constructor definition.
24   | We create a `Student` instance with named parameters as before, gpa first. When you name parameters, you can pass them in any order!

Note that I do not recommend putting parameters without default values after parameters with default values; this can cause confusion, but I wanted to point out the requirement for naming in this case.

### Initializers Can Access Primary Constructor Parameters
A really nice optimization is that property initializers have access to primary constructor parameters. This eliminates the need for many `init` blocks, as they often just initialize properties.

<pre theme="darcula" lines="true" match-brackets="true" data-highlight-only>
open class Person(name : String = "No Name") {
    var name : String = name // direct access to primary constructor parameter
}

// NOTE PARAMETER ORDER CHANGE!!!
class Student(name : String = "No Name", gpa : Float) : Person(name) { 
    var gpa : Float = gpa // direct access to primary constructor parameter
}
</pre>

Wow! Code reduction! Love it!

Looking at lines 2 and 7, we reference the primary constructor parameters in the property initializers. That removes the need for a dummy initializer value, and gets rid of the `init` blocks. Nifty! But the next step is the biggie... (Side note - my grandmother on my mom's side always hated the word "biggie" for some reason. Trips to Wendy's were always accompanied by her cringing...)

### Define Properties in the Primary Constructor
Here's one of my favorite things about Kotlin. If your constructor is passing in values that are directly used to initialize properties, you can define the properties _directly_ in the constructor. It's easiest to understand this in an example...

<pre theme="darcula" lines="true" match-brackets="true" data-highlight-only>
open class Person(var name : String = "No Name")

class Student(name : String = "No Name", var gpa : Float) : Person(name)
</pre>

Check out the terseness, but still _very_ readable. (I'd say even more readable now, as you no longer have the duplication and required explicit association of the constructor parameters and properties.)

By adding `var` or `val` in front of a primary constructor parameter, you

   - Define a property for the class
   - Initialize the property to the value passed in (or default value)

Line 1 defines the `name` property of `Person` and line 3 defines the `gpa` property of `Student`.

Because `Student` inherits `name` from `Person`, we don't add `var` or `val` to `name` in the `Student` constructor; it's just a normal constructor parameter that we pass on to the superclass.

Because these are `var` properties, we can modify them the same way we did before. We could also make them `val` properties, in which case, once the value has been set at creation time, we cannot change it.

You may have noticed that we no longer have a body (`{ ... }`) for these classes. If everything you need to define is in the constructors, you don't need a body and can remove the curly braces!

Property Subsets in Constructors
----------------------------------
One last thought before I pass out or my fingers refuse to keep typing...

Suppose we have several properties, some of which must be specified together or not at all.

For example, let's define a `Section` class that defines text in a document that may have a header and footer:

<pre theme="darcula" lines="true" match-brackets="true" data-highlight-only>
open class Section(
	val text : String, 
	val header : String = "", 
	val footer : String = "")
</pre>

I'd like to impose a restriction such that if we specify a header or footer, the other must also be specified.

We could add a check in the intializer...

<pre theme="darcula" lines="true" match-brackets="true">
open class Section(
	val text : String, 
	val header : String = "", 
	val footer : String = "") {

	init {
		if (header.isEmpty() != footer.isEmpty()) {
			throw IllegalArgumentException("Header and Footer must both be specified, or neither specified")
		}
	}
}

fun main() {
	val section = Section("Some text", "Section 1")
}
</pre>

Try running this and we'll get the exception.

But... I really prefer to catch things at compile-time when possible. So I'd really like to have constructors that either require _neither_ header nor footer, _or both_ header and footer. So we add a secondary constructor and tweak the primary:

<pre theme="darcula" lines="true" match-brackets="true">
open class Section(
	val text : String, 
	val header : String, 
	val footer : String) {

	constructor(text : String) : this(text, "", "")
}

fun main() {
	val section1 = Section("Some text")
	val section2 = Section("Some text", "Section 1", "End of Section 1")
	val section3 = Section("Some text", "Section 1") // will not compile
}
</pre>

Our primary constructor now requires both header and footer, and the secondary only requires the text, passing the default values to the primary. Perfect!

But what if we have parameters that create more complex groupings?

Let's create a very-poorly-designed class to represent a location (very-poorly-designed because we should use inheritance [and later, Kotlin's sealed classes!]). Start with

<pre theme="darcula" lines="true" match-brackets="true" data-highlight-only>
open class Location(
	val lat : Double,
	val lon : Double,
	val street : String,
	val city : String,
	val state : String,
	val zip : String)
</pre>

Here we want to have _either_ lat/lon, _or_ street/city/state/zip. (Yes... this is crazy gross, but I'm tired and this demonstrates the concept and my brain won't think of another example right now so there).

So we try

<pre theme="darcula" lines="true" match-brackets="true">
open class Location(
	val lat : Double,
	val lon : Double,
	val street : String,
	val city : String,
	val state : String,
	val zip : String) {

	constructor(lat : Double, lon : Double) : this(lat, lon, "", "", "", "")
	constructor(
		street : String,
		city : String,
		state : String,
		zip : String) : this(0.0, 0.0, street, city, state, zip)
}

fun main() {
	// good
	val location1 = Location(39.149810, -76.911257)
		// (sing it with me... "My Harris Teeter")

	// good
	val location2 = Location("123 Sesame St", "New York", "NY", "10001")

	// uh oh...
	val location3 = Location(39.149810, -76.911257, "123 Sesame St", "New York", "NY", "10001")
}
</pre>

The primary constructor is `public` by default. In this case, we really don't want that. So let's make it private.

<pre theme="darcula" lines="true" match-brackets="true">
open class Location private constructor(
	val lat : Double,
	val lon : Double,
	val street : String,
	val city : String,
	val state : String,
	val zip : String) {

	constructor(lat : Double, lon : Double) : this(lat, lon, "", "", "", "")
	constructor(
		street : String,
		city : String,
		state : String,
		zip : String) : this(0.0, 0.0, street, city, state, zip)
}

fun main() {
	// good
	val location1 = Location(39.149810, -76.911257)
		// (sing it with me... "My Harris Teeter")

	// good
	val location2 = Location("123 Sesame St", "New York", "NY", "10001")

	// uh oh...
	val location3 = Location(39.149810, -76.911257, "123 Sesame St", "New York", "NY", "10001")
}
</pre>

A little ugly, but poof! Private primary constructor. I haven't had to do this _too_ often, but I have had to do it...

Lots more to cover in later articles, but it's after midnight and I need some sleep... More soon!

