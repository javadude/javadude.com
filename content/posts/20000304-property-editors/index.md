---
title: Creating JavaBean Property Editors
aliases:
- /articles/propedit/index.html
- /articles/propedit

categories:
- Articles

date: "2000-03-04"

description: How and why you should use JavaBean accessor (get/set) methods.

tags:
- java
- beans
- design

---

Property editors are a way to make your bean more user-friendly.  Bean builder tools can determine quite a lot about your bean just by looking at it and its corresponding BeanInfo class, but they usually have only very simple methods of editing the properties in the bean.

<!--more-->

Many properties can be represented as a String, number or Color, and most Bean design tools have built-in editors for these types.  But if a property is another type of object, you must tell the design tool how to edit it.

The JavaBean spec defines a PropertyEditor interface in the java.beans package.   This interface has several ways of reading and writing information about a property.  This tutorial will walk you though several examples, from very simple String\-validation editors to more complex GUIs to define properties.

Unfortunately, the documentation available on property editors is rather sparse.   Until now, that is...

All source code is available for download in zip (Windows) and tar.gz (Unix) formats under #Source Code at the end of the article.

You should be able to use any development environment that supports JavaBeans to work with the code in this article.  Note that all code has been tested under VisualAge for Java 1.0, Visual Cafe for Java 2.1 and the November release of the BeanBox.   VisualAge and VisualCafe both performed as expected.

The BeanBox acts unexpectedly in a few cases.  I have checked with the folks at JavaSoft to see if these are bugs in the BeanBox and they informed me that they are.  The problems noted in BeanBox are:

*   If you provide a non-null return from `getAsText()` and provide a non-null `getCustomPropertyEditor()`, the BeanBox gives you no way of accessing the custom property editor GUI.
*   If you only provide `getAsText()`, `setAsText()`, `getValue()`, `setValue()` and `getJavaInitializationString()`, the BeanBox will ignore the property changes.   (It seems like the BeanBox requires PropertyChangeEvents in all cases, which is not required by the way I interpret the JavaBeans spec.)

## Making Life Easier

Before getting into the actual property editors, let's define a convenience "adapter" class for the PropertyEditor interface.  This adapter will provide default behaviors for all the PropertyEditor methods, and we can use it as a convenient superclass for the rest of our property editors.   We'll discuss the methods in this interface as we proceed through the article.

Adapters make life a bit easier for us because we don't have to define all the methods ourselves; we just need to override the ones that we really care about.

	// A simple adapter class for interface java.beans.PropertyEditor
	package com.magelang.adapter;

	import java.beans.PropertyEditor;
	import java.beans.PropertyChangeListener;
	import java.awt.Rectangle;
	import java.awt.Graphics;
	import java.awt.Component;

	public class PropertyEditorAdapter implements PropertyEditor {
		public void addPropertyChangeListener(PropertyChangeListener listener) {}
		public void removePropertyChangeListener(PropertyChangeListener listener) {}
		public void setAsText(String text) throws IllegalArgumentException {}
		public void setValue(Object value)     {}
		public void paintValue(Graphics gfx, Rectangle box)  {}
		public String    getAsText()                   {return null;}
		public Component getCustomEditor()             {return null;}
		public String    getJavaInitializationString() {return null;}
		public String[]  getTags()                     {return null;}
		public Object    getValue()                    {return null;}
		public boolean   supportsCustomEditor()        {return false;}
		public boolean   isPaintable()                 {return false;}
	}

Note that the JDK provides a class `java.beans.PropertyEditorSupport` that is very similar to this adapter (it adds better default support for property change notification and `getAsText`/`setAsText`.  I present this simpler adapter so we can avoid talking about property change events until later.  When you're really creating a property editor, you may want to use `java.beans.PropertyEditorSupport`.

The negative angle to using this approach is that your property editor cannot be directly included in another GUI, such as a Customizer.  If you want to use a property editor as a general bean as well as only a property editor, you can either:

*   Subclass your property editor from some component class (such as `Panel`) directly, and implement all of the above methods in your property editor, or
*   Create a wrapper bean for your property editor that is a component, and displays the property editor's GUI (you can get the GUI by calling `getCustomEditor()` from your wrapper...)

Note that if you extend Panel or another GUI component, you don't need to define the `addPropertyChangeListener` or `removePropertyChangeListener` methods as they are defined in `java.awt.Component`.

##Persistence in Bean Builders

It is very likely that once a user has set properties for bean, they'd like to see those properties appear in your property editor the next time they bring up the property sheet.  To provide this capability, property editors can define `setValue()` and `getValue()`.  Strictly speaking, you're not require to provide real implementations for these methods.  But if you don't, you may find some annoyed users at your door the day after release...

`public void setValue(Object value)` gives the bean builder a way to tell the property editor the current value of the property being edited.  For many fields this value will initially be null, but as the user edits the properties, the bean builder will store the values and pass them to the property editor when the property sheet is displayed.   The property editor can look at this value _but it should not directly modify it_!   The property editor should create a new object to hold the value for the property, and return that new object when the bean builder asks for it.

`public Object getValue()` provides a way for a bean builder to ask for the property value it will store with the visual design information.  This value will be kept and passed to `setValue()` any time the property sheet for the bean instance is opened.  The property editor is responsible for creating an object containing the value for the property, and returning that object.

Starting Simple: String Editors
-------------------------------

All Bean design tools will have some way of displaying a text field to get a String property value.  There is no validation done on this text field.

As our first property editor, we'll define a Color editor that has the user type in a color name and verify that the color name is one of the "basic" colors.

String property editors are fairly simple to program.  Interaction between a String property editor and the bean builder works as follows

*   The bean builder tells the property editor the current value of the property by calling the `setValue()` method
*   The bean builder asks "what String should be displayed" by calling the `getAsText()` method.
*   When the user changes the String value, the bean builder tells the property the new value by calling the `setAsText()` method.
*   The bean builder asks for the object value to store for the property by calling the `getValue()` method.
*   The bean builder asks for the code that will initialize the property by calling `getJavaInitializationString()`

One thing to note here: the bean builder is handling the interaction with the user, and informs the property editor when changes are made.  The property editor does not need to worry about informing anyone of the property being changed.  (This will become an issue when our property editor handles the user interaction and needs to inform the bean builder that the property value has changed.)

Our implementation of this simple property editor looks as follows.  Note that we subclass PropertyEditorAdapter so we only need to define the five methods mentioned above.

	package coloredit;

	import java.awt.Color;
	import com.magelang.adapter.PropertyEditorAdapter;

	// An overly simple Color editor.  The user types in the textual
	// name of a color and this class verifies that it is a basic color
	// name.  If it isn't, the color is set back to white.

	//Demonstrates use of the setAsText() and getAsText() methods.
	public class ColorEditor1 extends PropertyEditorAdapter {
		// Used to validate the color name
		//  (this property is protected to make the second 
		//   editor a bit easier...)
		protected static String colorNames[] = {
			"white", "lightGray", "gray", "darkGray", "black", 
			"red", "pink", "orange", "yellow", "green", 
			"magenta", "cyan", "blue"
		};

		// Used to map a color name to a Color object
		private static Color  colors[] = {
			Color.white, Color.lightGray, Color.gray, Color.darkGray, 
			Color.black, Color.red, Color.pink, Color.orange, 
			Color.yellow, Color.green, Color.magenta, 
			Color.cyan, Color.blue
		};

		// The currently-selected color (start with white)
		private int selected = 0;
		
		//Tells the bean builder the name of the current color 
		public String getAsText() {
			return colorNames[selected];
		}

		// Allows the bean builder to tell the property editor the value
		// that the user has entered
		public void setAsText(String text) throws IllegalArgumentException {
			for(selected = 0;
				selected < colorNames.length &&
				!colorNames[selected].equals(text);
				selected++);
			if (selected == colorNames.length)
				selected = 0;
		}

		// Tells the bean builder the value for the property
		public Object getValue() {
			return colors[selected];
		}

		// Allows the bean builder to pass the current property
		// value to the property editor
		public void setValue(Object value) {
			selected = 0;
			if (value != null)
				for(int i=0; i<colors.length; i++)
					if (value.equals(colors[i])) {
						selected = i;
						break;
					}   
		}

		// Get the initialization code for the property
		public String getJavaInitializationString() {
			return "java.awt.Color." + colorNames[selected];
		}
	}  


This property editor converts the passed-in Color value to a String representing the color name, validates that name whenever it is set, and converts it back to a Color object.

Using the Property Editor
-------------------------

There are two ways you can use property editors:

*   To provide editing capabilities for a specific property in a bean
*   To provide editing capabilities for all properties of a given type in the bean development environment

This article will concentrate on the first use.  A later article will discuss setting up a property editor as the default editor for a given type.

To associate a property editor with a bean property, you need to define a BeanInfo class for the bean.  Of course this means that we need a bean, so let's define a simple Panel subclass that has a "color" property whose implementation is to set/get the Panel's background color. (We will leave the background property intact so operation of this property editor can be compared to normal Color editor operation in the bean builder you are using.)

	package coloredit;

	import java.awt.Color;
	import java.awt.Panel;

	// A Simple Panel extension that we'll use to test our
	// property editor 
	public class ColorBean extends Panel {
		// note that there is no real "color" field;
		// a property can be strictly algorithmic...
		public Color getColor() {return getBackground();} 
		public void setColor(Color color) {setBackground(color);}
	}  

The BeanInfo class for this is big and hairy, and discussion of BeanInfo is out of the scope of this article.  (I'd advise you to get a tool, such as VisualAge for Java, that generates the BeanInfo class for you...) The following is the relevant part of the ColorBeanBeanInfo class that sets up the property editor.  The full BeanInfo class (as generated by VisualAge for Java) is available in the source distribution.

	package coloredit;

	// Relevant parts of the ColorBeanBeanInfo class 
	// Note that exception handling has been removed 
	// and the relevant parts
	// have been simplified...

	import java.beans.PropertyDescriptor;
	import java.beans.SimpleBeanInfo;
	import java.lang.reflect.Method;
	import java.awt.Color;
	import coloredit.ColorEditor1;

	public class ColorBeanBeanInfo extends SimpleBeanInfo {
		public PropertyDescriptor colorPropertyDescriptor() {
			// get the "get" method
			Class aGetMethodParameterTypes[] = {};
			Method aGetMethod =
				getBeanClass().getMethod("getColor", 
					aGetMethodParameterTypes);

			// get the "set" method
			Class aSetMethodParameterTypes = {Color.class};
			Method aSetMethod =
				getBeanClass().getMethod("setColor", 
					aSetMethodParameterTypes);

			// define the property descriptor
			PropertyDescriptor aDescriptor =
				new PropertyDescriptor("color", aGetMethod, aSetMethod);
			// Set a property editor
			aDescriptor.setPropertyEditorClass(ColorEditor1.class);
			return aDescriptor;
		}

		public PropertyDescriptor[] getPropertyDescriptors() {
			PropertyDescriptor aDescriptorList[] = 
				{colorPropertyDescriptor()};
			return aDescriptorList;
		}

		// and more BeanInfo stuff...
	}  

When you use the ColorBean when visually designing a new bean, you can bring up the property sheet for the new instance of the ColorBean.  Initially, the value for "color" will be "white".  If you change it to one of the valid color names, the change will stick.  If you mistype, the color value becomes "white" again.

Making the Property Editor a Bit Nicer...
-----------------------------------------

Because we have a (small) finite set of values, a nicer user interface would be to give the user a list of choices.  This is easily accomplished by adding a getTags() method to our color editor.  If getTags() returns a null (which is the default that we defined in PropertyEditorAdapter) the bean builder will just present a TextField to get the value.  If getTags() returns an array of Strings, the bean builder will present a Choice component with those String values.  So we can extend our ColorEditor as follows:

	package coloredit;

	public class ColorEditor2 extends ColorEditor1 {
		public String[] getTags() {
			return colorNames;
		}
	}  

This is possible because we defined colorNames as a protected field (even before we thought of this extension ;)

If we change the ColorBeanBeanInfo definition to use ColorEditor2, the property can now be edited by the user selecting the color choice in the property sheet.

Taking Control of the Property Edit
-----------------------------------

Up until this point, we've let the bean builder tool handle the interaction with the user.  For simple properties that can be represented by Strings this is perfectly acceptable.  (For example, perhaps you want to have the user type a name and you want to make sure it's a valid word in the Dictionary.  Your property editor could be similar to ColorEdit1, but do a spell-check on the value instead of checking the color.)  For more complex properties, such as Colors, we really want something more, well, colorful...

Let's take a small step toward a better property editor.  We'll present a Choice component with the color names.  This will require the following:

*   We need to create a simple GUI (the Choice component)
*   We need to tell the bean builder that we provide the GUI
*   We need to pass the GUI to the bean builder
*   We need to tell the bean builder when the property has changed

Let's look at the last item first: we need to tell the bean builder when the property has changed.  The PropertyEditor interface requires methods addPropertyListener() and removePropertyListener(), which let other classes ask us to inform them when a property changes.  The other class, in this instance, is the bean builder itself.

Let's take our PropertyEditorAdapter and extend it to add the property change notification support we need.  The JDK provides a support class, java.beans.PropertyChangeSupport, that will help us.  All we need is:


	package com.magelang.adapter;

	import java.beans.PropertyChangeSupport;
	import java.beans.PropertyChangeListener;
	
	// A default implementation of PropertyEditor Adapter 
	// that provides support for property listeners. 
	// This will be used for any property editors that 
	// perform their own interaction with the user to 
	// change the property value.
	public class PropertyEditorChangeAdapter extends PropertyEditorAdapter {
		private PropertyChangeSupport propertyChangeSupport =
			new PropertyChangeSupport(this);
		public void addPropertyChangeListener(PropertyChangeListener listener) {
			propertyChangeSupport.addPropertyChangeListener(listener);
		}
		public void removePropertyChangeListener(PropertyChangeListener listener) {
			propertyChangeSupport.removePropertyChangeListener(listener);
		}
		protected void firePropertyChange(Object oldValue, Object newValue) {
			propertyChangeSupport.firePropertyChange(null, oldValue, newValue);
		}
	}  

We define addPropertyChangeListener() and removePropertyChangeListener() to delegate their tasks to an instance of PropertyChangeSupport. We define an extra "helper" method, firePropertyChange(), to make life a bit easier for our derived property editors.

Now we'll take our original ColorEdit1 and change its superclass to PropertyEditorChangeAdapter.  We'll also change its name to ColorEdit3 to keep things clear.  After a few more changes, our new property editor looks like (changes to ColorEditor1 are in bold):

	package coloredit;

	import java.beans.PropertyChangeListener;
	import java.awt.Choice;
	import java.awt.Component;
	import java.awt.Color;
	import java.awt.event.ItemListener;
	import java.awt.event.ItemEvent;
	import com.magelang.adapter.PropertyEditorChangeAdapter;

	// This color editor will display a custom property editor and
	// allow the user to select a color name from a list.
	public class ColorEditor3 extends PropertyEditorChangeAdapter implements ItemListener {
		private static String colorNames[] = {
			"white", "lightGray", "gray", "darkGray", 
			"black", "red", "pink", "orange", "yellow",
			"green", "magenta", "cyan", "blue"
		};

		private static Color  colors[] = {
			Color.white, Color.lightGray, Color.gray,
			Color.darkGray, Color.black, Color.red, 
			Color.pink, Color.orange, Color.yellow, 
			Color.green, Color.magenta, Color.cyan, Color.blue
		};

		private int selected=0;
		private Choice myGUI;
		
		public String getAsText() {
			return colorNames[selected];
		}
    
		public Component getCustomEditor() {
			if (myGUI == null) {
				myGUI = new Choice();
				for(int i = 0; i < colorNames.length; i++)
					myGUI.addItem(colorNames[i]);
				myGUI.addItemListener(this);
			}   
			return myGUI;
		}

		public String getJavaInitializationString() {
			return "java.awt.Color."+colorNames[selected];
		}
    
		public Object getValue() {
			return colors[selected];
		}
    
		public void itemStateChanged(ItemEvent e) {
			setAsText((String)e.getItem());
		} 
    
		public void setAsText(String text) throws IllegalArgumentException {
			Object oldValue = getValue();
			for(selected = 0;
				selected < colorNames.length &&
				!colorNames[selected].equals(text);
				selected++);
			if (selected == colorNames.length)
				selected = 0;
			Object newValue = getValue();
			firePropertyChange(oldValue, newValue);
		}
    
		public void setValue(Object value) {
			selected = 0;
			if (value != null)
				for(int i=0; i < colors.length; i++)
					if (value.equals(colors[i])) {
						selected = i;
						break;
					}   
		}
    
		public boolean supportsCustomEditor() {
			return true;
		}
	}  


So what is it doing now?

First, we define supportsCustomEditor() to return true (recall that the default implementation in PropertyEditorAdapter returns false).   This method tells the bean builder that our property editor will have its own GUI.   The bean builder will then call getCustomEditor() to retrieve our GUI component that does the editing.  The bean builder will display this component in a dialog.

Our implementation of getCustomEditor() defines our simple GUI: a Choice component that is loaded with the color names.   To make this component useful, we need to register an ItemListener with it.  Because some of the bean design tools do not yet support inner classes (such as VisualAge for Java), we've made the property editor an ItemListener to catch the ItemEvent.  The action we perform when the item changes is to just set the text value in our property editor.

The last change we made was to fire a PropertyChangeEvent when the text value of the property changes.  This is accomplished by adding the firePropertyChange call to setTextValue().  firePropertyChange() checks to see if the old and new values are different, and, if so, fires off a PropertyChangeEvent to all PropertyChangeListeners (which in this case is only the bean builder.)

If you use this ColorEditor in a bean builder, many builder will show a small "..." button with the property in the property sheet.  Clicking this button will bring up a dialog with our Choice component in it.

A More Natural Interface
------------------------

The interface still leaves something to be desired.  What does "cyan" look like?  Why don't we provide some buttons with the colors as their background?   (Note: there are some versions of the JDK that have a bug in AWT's Button class that does not paint the background color of the Button.  The current version of VisualAge for Java contains this AWT bug.)

The interface between the property editor and the bean builder remains _exactly the same_.  All we are doing is changing our GUI a bit.  Differences between this editor and ColorEditor3 are in bold.

	package coloredit;

	import java.beans.PropertyChangeListener;
	import java.awt.GridLayout;
	import java.awt.Panel;
	import java.awt.Button;
	import java.awt.Component;
	import java.awt.Color;
	import java.awt.event.ActionListener;
	import java.awt.event.ActionEvent;
	import com.magelang.adapter.PropertyEditorChangeAdapter;

	// This color editor will display a custom property editor
	// and allow the user to select a color button.
	public class ColorEditor4 extends PropertyEditorChangeAdapter implements ActionListener {
		private static String colorNames[] = {
			"white", "lightGray", "gray", "darkGray", 
			"black", "red", "pink", "orange", "yellow",
			"green", "magenta", "cyan", "blue"
		};
		private   static Color  colors[] = {
			Color.white, Color.lightGray, Color.gray,
			Color.darkGray, Color.black, Color.red, 
			Color.pink, Color.orange, Color.yellow, 
			Color.green, Color.magenta, Color.cyan, Color.blue
		};
		private int selected=0;
		private Panel myGUI;
		public void actionPerformed(ActionEvent e) {
			setAsText(((Button)e.getSource()).getLabel());
		} 
    
		public String getAsText() {
			return colorNames[selected];
		}
    
		public Component getCustomEditor() {
			if (myGUI == null) {
				myGUI = new Panel(new GridLayout(0,2));
				for(int i = 0; i < colorNames.length; i++) {
					Button b = new Button(colorNames[i]);
					b.setBackground(colors[i]);
					b.addActionListener(this);
					myGUI.add(b);
				} 
			}   
			return myGUI;
		}
    
		public String getJavaInitializationString() {
			return "java.awt.Color."+colorNames[selected]; 
		}
    
		public Object getValue() {
			return colors[selected];
		}
    
		public void setAsText(String text) throws IllegalArgumentException {
			Object oldValue = getValue();
			for(selected = 0;
				selected < colorNames.length &&
				!colorNames[selected].equals(text);
				selected++);
			if (selected == colorNames.length)
				selected = 0;
			Object newValue = getValue();
			firePropertyChange(oldValue, newValue);
		}
    
		public void setValue(Object value) {
			selected = 0;
			if (value != null)
				for(int i=0; i < colors.length; i++)
					if (value.equals(colors[i])) {
						selected = i;
						break;
					}   
		}
    
		public boolean supportsCustomEditor() {
			return true;
		}
	}  

All we have changed is the appearance of the GUI (and how we respond to events.)   If you are careful with how you interact with the bean builder, you can change the appearance of the GUI with very little effort.  In this case, our GUI is now a Panel with a 2-column grid of Buttons. Each Button has the color name and (unless the AWT background color bug is present) the color that will be selected.   When pressed, the color value is set.

The Final Touch
---------------

But what about that textual color name in the property sheet?  Shouldn't that appear as a color as well?  It can!

To make this possible, we need to drop the getAsText() method and implement the isPaintable() and paintValue() methods.  Instead of passing a text value for the property back to the bean builder, we're telling it that we can paint a value for the property, and providing a method to do that painting.  Our changes to the class are very simple:

*   remove the getAsText() method*   add two imports:Figure 9: Imports to add
    
		import java.awt.Graphics;
		import java.awt.Rectangle; 

*   add an isPaintable() method (recall that the default defined in PropertyEditorAdapter returned false):
    
		public boolean isPaintable() {return true;}
    
*   add a paintValue() method:Figure 11: paintValue() method
    
		public void paintValue(Graphics gfx, Rectangle box) {
			gfx.setColor(colors[selected]);
			// we use three 3-d rects to over-emphasize the 3-d effect
			// (this way you can be sure this editor is working...)
			gfx.fill3DRect(box.x+2,box.y+2, box.width-5, box.height-5, true);
			gfx.fill3DRect(box.x+3,box.y+3, box.width-7, boxheight-7, true);
			gfx.fill3DRect(box.x+4,box.y+4, box.width-9, box.height-9, true);
		} 

paintValue() is passed a graphics context into which its painting will be done, and a bounding Rectangle that tells us where we are allowed to draw within that graphics context.  _Make sure you respect that bounding box, or the property sheet graphics could end up looking pretty nasty!_  Our implementation for ColorEditor5 draws a raised box in the selected color.

PropertyEditorSupport
---------------------

In this article, we define the PropertyEditorAdapter and PropertyChangeAdapter to help demonstrate how property editors work. The Java class libraries include a class named java.beans.PropertyEditorSupport which provides this base functionality for you. I recommend that when you really implement property editors, you use property editor support.

Conclusion
----------

Property editors are a simple way to make your bean more user friendly.  They can provide simple textual validations, a GUI to assist the modification of the property and graphical representations of the property within the property sheet itself.

If you have any questions on how to get a property editor to work, feel free to email [Scott Stanchfield](mailto:scott@javadude.com) or one of the other Magi.

Source Code
-----------

All of the source code for this article is downloadable.  There are two formats you can choose:

*   [propedit.zip](propedit.zip) - a Windows-format zip file (note that files are formatted using \\r\\n for line separators)
*   [propedit.tar.gz](propedit.tar.gz) - a unix-format tar/gzip file (files use \\n for line separators)

 