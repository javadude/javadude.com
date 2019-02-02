---
title: Import on Demand is EVIL!
aliases:
- /articles/importondemandisevil.html

categories:
- Articles

date: "2004-05-22"

description: Don't use ".\*" on your imports!!!

tags:
- java
- language


---
# Introduction

There should _**never**_ exist any language feature such that _**adding**_ a new type to a referenced package can break your existing code. Import-on-demand is one such feature... And it's evil...

<!--more-->

For example:

	import a.b.*; // your package
	import x.y.*; // 3rd-party code

	class Foo {
		Bar bar;
		...
	}

Suppose that to start with, only `a.b.Bar` exists. You compile and check-in your code and all's fine.

This code has a [Bus Number](http://c2.com/cgi/wiki?BusNumber) of one...

A year later, after you've been hit by said bus and there have been some updates to the jars containing `a.b` or `x.y`, someone needs to make a mod. They check out the code and start cursing you because you had the nerve to check in code that doesn't compile.

But you _did_ check in compiling code! Turns out, the 3rd party lib you were using is now version 2 and includes class `x.y.Bar`. Worse, maybe Bar is similar to your Bar, making it even more difficult to determine which Bar you actually intended.

# The Big Bang

This is language design at its worst, and Sun made this blow up in everyone's face between JDK 1.1 and 1.2. In 1.1, there existed `java.awt.List`. Tons of folks wrote code that included

	import java.util.*;
	import java.awt.*;

I can't begin to count the number of code examples that contained those two lines). If you used List in your class, like

	List choices = new List();

Your code compiled fine in 1.1, but a runtime lib upgrade to Java 1.2  breaks your code!

BTW: Sun _knew_ and announced this would happen. They should have created `java.util.collections.List` instead of `java.util.List` to prevent it. It forced lots of people to change existing code for no gain.

I've seen and heard of this happening again and again and again. Explicit imports make this all go away.

When I taught Java for MageLang Institute (now [jGuru.com](http://www.jguru.com)), my compromise advice was always "use import-on-demand when developing, then fix it before checking in." I wrote a tool called ["Importifier](../tools/importifier/index.html)" for VisualAge for Java that did this expansion automatically. Fortunately, it's in [Eclipse](http://www.eclipse.org), so I didn't have to port it.

# Tools can help!

With today's tools (like [Eclipse](http://www.eclipse.org) and [IDEA](http://www.jetbrains.com/idea/)), there's _**never**_ an excuse to use import on demand. You can easily expand imports, removing any possibility of this nasty problem. Even better, [Eclipse](http://www.eclipse.org) at least  automatically adds the imports if you use code completion. I haven't written "import" when coding in nearly three years! (What I mean by this is that for anything other than simple demos, I let [Eclipse](http://www.eclipse.org) generate all of my imports just by using code completion.)

"Import on demand saves time" is not a valid argument. And if you're one of those "You're not a real programmer if you use an IDE" types, I truly pity your stubbornness and dedication to making life so much harder on yourself...

BTW: As for performance, there's a minor gain in speed _**at compile time**_ using explicit imports (the compiler doesn't have to check all "\*" imports to check which applies, but it's nearly unnoticeable). However, there is absolutely no difference at runtime. In case you don't know, all class name references are fully-resolved at compile time and hard-coded into the .class file. Of course this doesn't apply to reflection.
