---
title: Importifier
aliases:
- /tools/importifier/index.html
- /tools/importifier

categories:
- Tools

date: "2000-03-04"

description: Automatic Expansion of Java Import Statements

tags:
- java

---

Automatic Expansion of Java Import Statements

<!--more-->
# News

*	10/17/2018
	*	Updating my website. This tool is obsolete, and only runs in VisualAge for Java. I'm keeping it here for historical purposes. The same functionality (cleaning up imports) is available in most modern IDEs.

*   6/26/2001
    *   Fixed line separation characters. Now the proper line separator ('\\n', '\\r', or "\\r\\n") will be appended after each import statement. (Ummm... oops, silly me ;)
    *   Added checks that the source code compiles cleanly and is versioned. Note that you must now version classes before importifying them.
    *   Moved messages into the status dialog rather than in the VisualAge console.  
         
*   7/13/2000
    *   Important limitation reported! If you only reference finals in another class/interface, importifier will not add an import for that class. See the limitations section below. Don't know if I'll be able to do anything about this near-term, so you may have to add a few imports by hand when the compiler complains...  
         
*   3/12/2000
    *   Initial release of Importifier
    *   Status: Works pretty well, but I'd recommend you version your source _before_ using it, _just_ in case...  
        **_Note: This version of the tool only works inside VisualAge for Java._** I've written the code such that I should be able to easily create a command-line version for non VAJ users in a later release.

# Introduction

Java provides two forms of import statements:

*   **Explicit import** - You write an import statement like
    
    import java.awt.Button;
    
    which tells the compiler "when you see 'Button', I mean 'java.awt.Button'.  
     
*   **Import on demand** - You write an import statement like
    
    import java.awt.\*;
    
    which tells the compiler "when you see a name you don't recognize, check to see if it exists in package 'java.awt'"  
     

**_Import on demand is pure, unadulterated EVIL_**.

There are several problems with import-on-demand:

*   If you use a name that exists in more than one package, the Java compiler cannot determine which fully-qualified name to use. You need to specify the full qualification yourself.
*   If a name is currently unambiguous (exists in only one package that you specify via import-on-demand), someone could add the same name to another package and your program will no longer compile.
    *   _Note that this happened with the Java2 platform. Originally, the List class existed in package java.awt. In Java2, Sun added a new List class to the java.util package. If your program imports java.awt.\* and java.util.\*, and uses List, it will compile in JDK 1.1 but not in the Java 2 platform!_
*   If someone is reading your code, they need to check in each specified package to see which contains a referenced class Foo.

The importifier replaces all import statements with _exactly_ the import statements your program requires.

# How the tool works

_First, it is important that the code you want to importify be error-free._ The tool needs to examine the bytecode to determine the correct imports, as well as resolve ambiguities.

_Second, you must version your classes and interfaces before running importifier against them_. This is a safeguard, _just_ in case importifier decides to have a bad fur day.

Importifier starts by exporting the code you wish to importify. The tool exports .class files to a temporary directory, so it can parse the bytecode.

Next, importifier parses the bytecode to see which classes are used. This is significantly easier than parsing the source code to find out the same information.

After the tool knows which classes are used, it walks through the source code of the class definition to remove the old import statements and add the new, fully-qualified ones. In many cases this is straightforward, but in cases where the short name of the classes are ambiguous (as in List being in both java.awt and java.util in the Java2 platform), importifier examines your old imports to see which full name it should keep.

If the tool sees any inner classes listed in the .class file, it recursively parses the inner classes to determine all imports needed in the containing class.

# Distribution

Importifier is packaged as a tool for VisualAge for Java 3.02 and above. _**Importifier will not work on earlier versions of VisualAge for Java. Note that the latest release has not been tested on versions earlier than 3.5.3 but should work on 3.02.**_

## Installation for VisualAge for Java - Tool Packaging

To use Importifier in VisualAge for Java:

1.  Download [importifier.zip](importifier.zip) from this site.
2.  Unzip importifier.zip to the directory where VisualAge for Java is installed.  For example, if you installed VisualAge for Java in the c:\\IBMVJava directory were on your C:\\ drive, unzip the importifier.zip to C:\\IBMVJava directory. Note that VisualAge for Java version 3.5 and above are typically installed under _C:\\Program Files\\IBM\\VisualAge for Java_.
3.  Shutdown VisualAge for Java if it is running
4.  Start VisualAge for Java.

This will add the importifier tool to the Tools menu for classes, packages, and projects.

_**Note: If you are using VisualAge for Java 3.02 on Linux, you will need to download the tool api for linux. You can get this from VisualAge Developer Domain ([http://www.software.ibm.com/vadd](http://www.software.ibm.com/vadd)). The latest version of importifier has not been tested under Linux. Please let me know if you encounter any problems!**_

Use
---

## VisualAge for Java

To use Importifier (once it is installed)

*   Version the classes, packages, or projects that you want to importify. Note that the tool will check if the code is already versioned and abort if it isn't.
*   Select **Tools->Expand import statements** from the popup menu of selected classes, packages, or projects
*   The status dialog for the Importifier reports its progress.

## Command-Line

The importifier tool does not yet support command-line use. I will eventually release a version that includes command-line capability.

# Limitations

**_This tool will not add imports for classes/interfaces from which you only use final variables._**

Importifier reads the generated bytcode for the class being importified to figure out what imports are needed. If all you reference are constants (finals) in another class/interface, it won't see that class/interface because the java compiler resolves them, leaving no reference to that class/interface in the bytecode. Bummer...

IMHO, this is a design flaw in Java, but that's another issue... I'll have to see if there's some way I can fix it later, but don't know that I'll have time anytime soon. For now, you may have to add a few imports by hand after running importifier. It will still give you a great start toward freedom from import-on-demand, though.

**_This tool does not work on classes that exist in a default package!_** You should only use the tool on classes in a named package!

The tool does not change full-qualifications in the code to short names. I will add this capability in a later version of the tool.

You may need to clean up the code a bit after the tool is done, as it adds a bit of extra space. In many cases the space will be correct, but in some cases you may find there is an extra blank line between the imports and the class definition. _Note also that the resulting file might look a bit strange if you had comments on the same lines or near the old import statements._

**_The tool will only work if the code to importify is error free!_**

The tool currently creates import statements for classes in the same package as well as classes in package java.lang. A later version of the tool may provide options to omit these imports if desired.

# License

Importifier is free for _any_ use _other_ than selling it as a stand-alone product.

Note that Importifier is provided "as-is" without any warrantee or guarantee of suitability for any purpose. Scott Stanchfield cannot be held responsible for any damage caused by its use.

(In other words, use 'em for free at your own risk.)
