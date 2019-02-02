---
title: Autogut
aliases:
- /tools/autogut/index.html
- /tools/autogut

categories:
- Tools

date: "1999-10-24"

description: A VisualAge for Java plugin to create interfaces from classes

tags:
- java

---

A VisualAge for Java plugin to create interfaces from classes

<!--more-->

## News

*	10/17/2018
	* I'm converting my website to a new form. This tool is obsolete (as it is a VisualAge for Java plugin), but I wanted to keep it here for historical purposes

*   6/7/2003
    *   In case you didn't know, Eclipse has this function built in. To use it
        *   Select the name of a class in Eclipse
        *   Choose **Refactor->Extract Interface**

*   10/24/99
    *   Initial release of autogut
    *   Status: An initial release - not entirely robust, but should work pretty well. I need to add a bit more checking for situations like "project already exists in repository" and ask if you want to create it. I suggest for now that you always use the Browse buttons for project and package.  
        This release has not undergone a great deal of testing, but should not pose any risks to your existing data. It does not modify the 

# Introduction

VisualAge for Java provides some excellent tools for JavaBean component creation and maintenance. However, it is often important to create interfaces to abstract a bean, especially for use in design patterns such as Model-View-Controller (MVC).

In a normal MVC development cycle, you first create your interfaces, then implement them as concrete models. Conceptually, this is the right way to go. However, this can involve a lot of typing. Often, the model interface provides standard bound properties and custom event registration capabilities.

VisualAge for Java's BeanInfo editor creates JavaBean properties and events quite easily. We can leverage this by creating a concrete model _first_, then gutting it to create an interface.

AutoGut examines an existing class and creates an interface for all of its public methods. This provides a good initial interface that you can use for your model. You may need to do a bit of tweaking (AutoGut currently does not copy import statements, for example), but this is much simpler than writing the interface by hand.

# Distribution

I have just packaged autogut as a tool for VisualAge for Java 3.0. It will probably work in version 2.0 but I have not tested it there.[Tools](../index.html)

### Installation for VisualAge for Java - Tool Packaging

To use AutoGut in VisualAge for Java:

1.  Download [autogut.zip](autogut.zip) from this site.
2.  Unzip autogut.zip to the directory where VisualAge for Java is installed.  For example, C:\\IBMVJava.
3.  Shutdown VisualAge for Java if it is running
4.  Start VisualAge for Java.

This will add the autogut tool to the **Workspace->Tools** menu, as well as the Tools menu for classes.

# Source Code

If you're interested in the source code for this tool, you can download [Autogut-dat.zip](Autogut-dat.zip). Note that you _do not_ need the source code to use the tool. This is a zipped VisualAge for Java repository file. You will need to unzip this file and import the JavaDude AutoGut project from this repository.

# Use

## VisualAge for Java

To use AutoGut (once it is installed)

*   Select **Tools->Create Interface from Class** from the **Workbench** menu or from a class' popup menu
*   Specify the name of the class to gut (automatically selected if you had started AutoGut from a class' tool menu)
*   Specify the name of the interface you want to create
*   Specify the project and package where you want to create that interface. Note that if you pick a package that is in the workspace it will automatically select the project.

# Limitations

Currently the tool does not copy the import statements from the class. You may need to add them yourself.

The tool is not completely bulletproof yet, though it's in pretty good shape. Just be careful what you type.

# License

AutoGut free for _any_ use _other_ than selling it as a stand-alone product.

Note that AutoGut is provided "as-is" without any warrantee or guarantee of suitability for any purpose. Scott Stanchfield cannot be held responsible for any damage caused by its use.

(In other words, use 'em for free at your own risk.)
