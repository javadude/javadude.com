---
title: Using JavaBean Accessors
aliases:
- /articles/accessors.html

categories:
- Articles

date: "2000-03-04"

description: How and why you should use JavaBean accessor (get/set) methods.

tags:
- java
- language
- design

---
Accessors are a key ingredient to proper encapsulation.  The can restrict access to only read/write data, inform other objects of changes to data, and perform validation on new values being set.  Consistent accessor usage makes class modification easy and transparent.

<!--more-->

What are Accessors?
-------------------

An accessor is a method that allows read or write access to a piece of class or instance data.  These are commonly defined using the naming convention

*   get_VarName_() -- reads the data for class/instance variable _varName_
*   set_VarName_(_value_) -- sets the value for class/instance variable _varName_

By providing and using accessor methods to read/write data contained within a class, you provide stronger encapsulation for the data of that class.  This has several advantages, especially if you decide to change the implementation of the underlying data (or remove the data itself and replace it with a computation).

In addition, creating accessors for a private class/instance variable allow you to restrict access to read-only or write-only by providing _only_ a get or set method.

But That's Inefficient!
-----------------------

The most-common objection to use of accessors is "but that's another method call, and it will drastically slow down my program!"

Not any more!

The new HotSpot VM provides excellent run-time optimization.  Part of this optimization is automatic inlining of simple methods.  If you write accessors that just set and return a value, these will be inlined to simple assignment statements.   Accessors that do more may still be inlined, depending on their complexity.

Bottom line: set aside any worries about accessors causing a performance hit.

All Data **Must** Be Private!
-----------------------------

I could go on and on about this forever, but I'll try to keep it brief: All data defined in a class _**must**_ be _private_!

I may sound a bit like an "OO-purist" when I say this, but I view this from a practical standpoint, having been burned by not doing it...

Making data (by data I mean class and instance variables) non-private allows another class to change their values without the owner of that data knowing about the change.   If the owner doesn't know about the change:

*   the owner has no control over the value; it could be outside the intended range of the data
*   the owner cannot know the value has change, and cannot respond if needed
*   the owner cannot tell others that the value has changed
*   the owner cannot allow others to _veto_ the change

On top of these problems, if you ever decide to change your code to allow other classes to listen for changes, or restrict bounds on a variable, you would need to find every reference to the data and change it.  This can be very painful if the class were a public library that many people had used in their own code.

Even worse, you cannot change the implementation of the data without changing the users of that data.  For example, suppose you had an integer instance variable to keep track of some ID for a list node.  Perhaps you had done some research and decided that the memory cost of keeping that ID was more expensive than recomputing it the few times it was needed.  If ID were only available via an accessor, all you would need to do to affect the change is to remove the instance variable and change the getID accessor to compute the ID and return it.  Far simpler than creating the getID accessor and going back and changing every reference to x.id...

Privacy is the key to true encapsulation.  All manipulation of an object's state **_must_** be done _by the object itself!_

Once you use accessors to read/write class/instance data, that data is instantly available as a JavaBean property!

JavaBean Properties
-------------------

The Java Bean spec defines the entities described by get and set accessors as "properties".  I use the term _entities_ here because a property does not need to have any actual data associated with it.  (A get method could just perform a computation instead of returning a variable's value.)  By using the simple get/set naming convention, any tool that supports JavaBeans can determine which properties are available for a bean _just_ by looking at the names of the provided methods!

JavaBean properties come in four flavors:

*   simple properties
*   bound properties
*   constrained properties
*   indexed properties

Properties can be any combination of bound, constrained and indexed, or just "simple" properties.

### Simple Properties

These are basic properties defined by a JavaBean, and can be readable, writable or both.

### Bound Properties

This is where the real power of accessors shines.  A bound property is one that will dispatch an event any time it has changed.  By restricting write access of a property to its set method, you can have the set method fire PropertyChangeEvents to any object registered to listen for a change to that property!

### Constrained Properties

Taking the bound concept a bit further, we can tell listeners that a property has changed _and allow those listeners to veto the change!_  This works pretty much the same as a bound property, but if the listener doesn't agree with the change, he can throw a PropertyVetoException which the property's set method will respect and halt the actual data change.

### Indexed Properties

The JavaBeans specification provides for one more type of property.  Indexed properties are properties that appear like arrays.  You can read or write the entire group of property values with one get/set method, or you can read/write individual elements of that indexed property.  Again, the data does not have to be represented as an array; it could be stored in _any_ format, or there might not even _be_ data behind the property.

Use Accessors in the Class that Defines the Data!
-------------------------------------------------

A final note: _All_ references to class/instance data should be through accessors, _even_ if the reference is in the class that defines that data!   This centralizes access to the data, allows all changes to be known (including notification for bound and constrained properties.)

This may seem like a bit of overkill, but if you ever need to change how a property is implemented, you'll be glad you used accessors consistently.

