---
title: JavaOne 2000/2001/2002
aliases:
- /javaone/index.html
- /javaone

categories:
- Articles

date: "2000-03-04"

description: Some talks I gave at JavaOne in 2000, 2001 and 2002.

tags:
- java
- language
- design

---

Some talks I gave at JavaOne in 2000, 2001, and 2002.

<!--more-->

#JavaOne 2002

First, thanks to everyone who showed up at my talk, _Creating Custom JSP Tags._ At least that was the title until the legal folks got hold of it...

I've posted a non-JavaOne copy of the slides as well as code samples that can be used in Tomcat.

*   [Creating Custom JSP Tags.zip](Creating%20Custom%20JSP%20Tags.zip) - powerpoint slides
*   [customtag.zip](customtag.zip) - a zipped web application containing source of the sample code demonstrated during the lecture

## Using the Sample Code

### Trying the samples on your machine

The sample code has been tested under Tomcat 4.0.3. (You can obtain tomcat from [http://jakarta.apache.org](http://jakarta.apache.org/)).

To install the sample code:

1.  Download it using the above link.
2.  Unzip it into the webapps directory of your tomcat installation. This should create a customtag directory under webapps.
3.  Modify the conf/server.xml file in your tomcat installation to create a context for custom tag. You can add a line like
    
		<Context path="/customtag" docBase="customtag" debug="0" reloadable="true"/>
    
    in your server.xml file. Search for the other Contexts that are defined, such as the manager context, and place this new context there.  
         
4.  Restart your tomcat server.

Once you have the sample code installed, you can surf to any of the following URLs to view the samples. Note that these URLs assume a standard development installation of tomcat, listening at port 8080 on your local machine. If tomcat is installed elsewhere, you'll need to modify the URLs accordingly.

URLs for custom tag samples _on your machine_

*   [http://localhost:8080/customtag/header1.jsp](http://localhost:8080/customtag/header1.jsp)
    *   A simple JSP that uses a custom header tag for a hardcoded replacement.  
         
*   [http://localhost:8080/customtag/header2.jsp](http://localhost:8080/customtag/header2.jsp)
    *   Similar to header1, but this tag takes an attribute to allow simple customization of the replacement.  
         
*   [http://localhost:8080/customtag/ifsample.jsp](http://localhost:8080/customtag/ifsample.jsp)
    *   A JSP that conditionally includes some text based on parameters passed to it. Try
        
        *   [http://localhost:8080/customtag/ifsample.jsp?showName=true](http://localhost:8080/customtag/ifsample.jsp?showName=true)
        *   [http://localhost:8080/customtag/ifsample.jsp?showName=false](http://localhost:8080/customtag/ifsample.jsp?showName=false)
        
	as variations to see how this page behaves.  
         
        
*   [http://localhost:8080/customtag/listpeople.jsp](http://localhost:8080/customtag/listpeople.jsp)
    *   A JSP that uses a custom iterator tag to walk through data retrieved from a data source. Note that the data manager used for this example hardcodes the data for simpler setup. If you want to change it to use a database, you only need to change the data manager class.  
         
*   [http://localhost:8080/customtag/layout.jsp](http://localhost:8080/customtag/layout.jsp)
    *   A JSP that uses nested custom tags to provide AWT-like BorderLayout support.

## License

The presentation is Copyright (c) 2002, Scott Stanchfield, All Rights Reserved. It may not be used for commercial purposes without obtaining permission.

This sample code is free for _any_ use _other_ than selling it as a stand-alone product.

Note that the sample code is provided "as-is" without any warrantee or guarantee of suitability for any purpose. Scott Stanchfield cannot be held responsible for any damage caused by its use.

(In other words, use 'em for free at your own risk.)

#JavaOne 2001

First, thanks to all who attended my talk, _Effective Layout Management._ I'm posting a non-JavaOne copy of the slides here. Be sure to read the original article, _Effective Layout Management_. A link can be found on my [articles](../../articles) page.

**Note**: I originally planned to update the _Effective Layout Management_ article, but I have been approached by a publisher to write a book on the subject. I have not decided yet if I will actually write the book or not.

*   [Effective Layout Management.zip](Effective%20Layout%20Management.zip) - PowerPoint Slides of _Effective Layout Management_ presentation

The presentation is Copyright (c) 2001, Scott Stanchfield, All Rights Reserved. It may not be used for commercial purposes without obtaining permission.

# JavaOne 2000

First, thanks to everyone who showed up at the BOFs ("Birds of a Feather" sessions) that I hosted! I had a great time jabbering on about VisualAge for Java, Model-View-Controller, Actions, and layout managers. I'm glad we got a chance to let the IBMers answer some of your buring questions, and many of those answers were just what we're looking for!

If you have any questions on anything I talked about, please don't hesitate to email me at [scott@javadude.com](mailto:scott@javadude.com)!

## Model-View-Controller BOF ("MVC for You and Me")

As promised, you can download the slides for this talk here, as well as find some other interesting resources on MVC. Enjoy!

*   [MVC for You and Me.zip](MVC%20for%20You%20and%20Me.zip) -- PowerPoint slides for my MVC talk. If you don't have PowerPoint, you can download a free viewer from Microsoft.com. 
*   [Applying the Model-View-Controller Paradigm in VisualAge for Java](../vaddmvc1/mvc1.htm)  
    Introduction to the Model-View-Controller Paradigm and details on applying it visually in VisualAge for Java. _Note: This article has many specifics to VisualAge for Java, but the general concepts and the walkthrough demonstration can be useful to help you see how to apply MVC in other environments as well.  
    _ 
*   [Advanced Model-View-Controller Techniques](../vaddmvc2/mvc2.html)  
    A more general, conceptual article on some more advanced MVC topics. Much mess emphasis on VisualAge for Java, and includes examples of sorting, merging, and filtering Swing tables.  
     

## Actions: Experience and Speculation

In case you're interested in the Swing Action slides, you can access there here:

*   [actions.zip](actions.zip)

## Fun with Layout Managers

I've put the MaxMinLayout manager file here for you to look at. Note that it has some rough edges, and is only meant to represent a simple example of a custom layout manager. It's actually an example from my book, _[Effective VisualAge for Java](../../evaj/index.html)_, where I discuss how to integrate custom layout managers with VisualAge for Java.

*   [MaxMinLayout.zip](MaxMinLayout.zip)

## Look and Feel Guidelines: What do they mean to you?

My apologies to anyone who wanted to come to this BOF. I had a few issues come up during the first day at JavaOne and had to cancel it. Please be aware that Sun is working on version 2 of the Look and Feel Guidelines, and they're really interested in any feedback you may have on the initial guidelines.

