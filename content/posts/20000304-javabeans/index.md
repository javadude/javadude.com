---
title: The JavaBean Component Model
aliases:
- /articles/javabean.html


categories:
- Articles

date: "2000-03-04"

description: How and why you should use JavaBean accessor (get/set) methods.

tags:
- java
- beans
- design

---

The basics of the JavaBean Component Model

<!--more-->

(Note - this article is quite old... I only had time to reformat it, so please forgive use of "Vector", though I'm not sure why I used it... I have anonymous inner classes in here, so `java.util.List` existed... I assume it was just code evolution at some point...)

JavaBean components, or simply beans, are classes that are proud of what they are, and they let the world know it. Using certain naming conventions and possibly a helper class, a bean identifies its attributes and behavior to any interested party. Bean-builder tools or other applications use that knowledge to customize beans and glue them together through event registration.

You have most likely seen this type of behavior in the form of bean-builder tools. You drag and drop GUI components in a design area, change their attributes, and specify interactions between them. However, you might wonder how these tools can know so much about the components. In fact, these tools are much simpler than they appear, thanks to the JavaBean component model.

The power of beans is not limited to designing user interfaces. A bean can be a nonvisual component, such as a data structure or a proxy for a remote database. Bean-builder tools manipulate beans, helping to build applications without requiring you to write code. Moreover, beans are not only useful in application design tools; they can prove useful in run-time environments as well.

What Makes a Class a Bean?
--------------------------

All beans must meet two requirements:

*   They must be public classes (so any class can inquire about it).
*   They must support persistence of their state.

The bean specification enables this persistence through the java.io.Serializable or java.io.Externalizable interfaces. This requirement enables an application such as a bean-builder tool to restore a "pickled" instance of a bean, an instance created before running the application.

If you plan to always restore the bean from its serialized state, no other requirements must be met. However, it is generally desirable to create new instances of a bean by using a new expression. To enable this, the bean must provide a public no-argument (otherwise known as default) constructor.

_Tip: Many bean-builder tools, including VisualAge for Java, interpret the bean specification as requiring public no-argument constructors. Others require persistence. Because of this, always make your beans persistent and define a public, no-argument constructor. If you do not make your bean serializable, it may not be usable in other bean-builder tools._

Of course, simply making a class persistent and possibly providing a public no-argument constructor does not make the bean terribly interesting. For a bean to be useful, it should provide information about its attributes and behavior.

Bean Features
-------------

The bean specification defines three types of bean features. Features are aspects of a bean that might prove interesting to another application or class, describing the state and behavior of the bean. The three types of bean features are:

*   **Properties**. Exposed pieces of the bean’s state
*   **Event Sets**. A stimulus from the bean, signaling that something, like a changes of state has occurred. The bean broadcasts this information to its listeners
*   **Methods**. Public methods, callable from other beans or applications

All bean features must be public! This allows any other bean to inspect or modify properties, register itself as an event listener, or call provided methods. This does not mean that the actual data inside the class needs to be public, nor do all methods in the class need to be public. Only the parts of a bean that describe its features must be public.

Through use of simple naming conventions, a bean enables applications to determine which features it supports. Tools discover bean features solely through the names of the public methods defined in a bean class. A beans-enabled tool can use the Java language reflection API to determine which public methods a class defines; the tool can also determine available bean features based on the names of those methods. Some beans also provide an explicit class called a BeanInfo class that describes features in more detail, but this is not necessary. This additional information can enhance design-time use of the bean, but has no affect on the runtime behavior of the bean.

Properties
----------

Properties describe part of the state of its bean, usually information that can distinguish different instances of a bean. There are two types of properties: simple and indexed.

_Note: A property can be either simple or indexed; it cannot be both._

### Simple Properties

Simple properties act like individual bits of data, describing the state of a bean. You define simple properties using sets of methods that match the following patterns:

```java
public TypeName getPropertyName()
public boolean isPropertyName()
public void setPropertyName(TypeName value)
```

Note: You can only define the `isPropertyName()` method for boolean properties.

`TypeName` is the name of a class or primitive type, and `PropertyName` is the name of the property. The presence of a get or is method defines a readable property. A set method defines a writeable property. The property name is actually a lower-cased version of whatever follows the get or set in the method declaration. For example, suppose a bean defines the following method:

```java
public Color getEyeColor()
```

This defines a readable property named eyeColor of type `java.awt.Color`.

Note that there can be both a get and is method for a boolean property; the is method is allowed for readability. Normally, you define the get and set methods, but sometimes the get method doesn't read naturally. For example

```java
if (list.isEmpty()) {...}

// is more readable than

if (list.getEmpty()) {...}
```

The beauty of this model is its simplicity. Many programmers already use get and set methods to access their data, so defining properties is no different from their current style.

A tool that recognizes these get and set methods can present a list of properties to a user, enabling the user to specify property values. The tool generates code to pass the specified values to the set methods.

Because the tool is only querying the names of the methods, there does not need to be any actual data corresponding to the property. A property could simply be a computation or an alternate means to validate and set other data inside the class. An `isEmpty()` method might simply count items in a referenced list, or a getMaximumValue() method might return an integer such as five.

### Indexed Properties

Indexed properties are an extension of simple properties that allow multiple values for that property. The property acts like an array, even if there is no actual data for that property or if the data is stored in a non-array structure.

You define indexed properties using four methods:

```java
public TypeName[] getPropertyName()
public TypeName getPropertyName(int index)
public void setPropertyName(TypeName[] value)
public void setPropertyName(int index, TypeName value)
```

Recognizing these methods, a bean-builder tool can present a (possibly editable) list of values for setting the property.

### Bound and Constrained Properties

While these names may conjure images of Harry Houdini, bound and constrained properties have nothing to do with ropes, shackles, or chains. (If you're thinking something other than Harry Houdini, remember that this is a family show...)

Bound and constrained properties fire events that report any modification to their state. Bound properties notify registered listeners after their state has changed, while constrained properties notify listeners before they change, allowing those listeners to veto the change. You can bind and constrain simple or indexed properties.

Bound or constrained properties greatly enhance the flexibility of a bean. Bound properties enable you easily to keep property values in different beans synchronized. VisualAge for Java provides techniques to take advantage of bound properties. Constrained properties allow you to plug in validation, in the form of listener objects, rather than hardcoding that validation into the property code itself.

### Bound Properties: Reporting Property-State Changes

(NOTE: This is an implementation of the Observer Pattern...)

A bound property fires a `PropertyChangeEvent` (defined in package `java.beans`) when its state changes. `PropertyChangeListener`s register themselves with the bean to receive notification of those changes. The `java.beans` package provides support for this processing through its `PropertyChangeSupport` class. For example, consider a simple property phoneNumber, defined as follows:

```java
private String phoneNumber;

public void setPhoneNumber(String phoneNumber) {
	this.phoneNumber = phoneNumber;
}

public String getPhoneNumber() {
	return phoneNumber;
}
```

_Note: Sometimes the statement_

```java
this.phoneNumber = phoneNumber;
```

_can be quite confusing to read, especially when first learning the Java language. We use it here because it is and accepted common practice. The `this` qualification on the first `phoneNumber` means, "I'm talking about the **instance** variable `phoneNumber`." This is necessary to distinguish it from the current active use of `phoneNumber`, the **parameter** `phoneNumber`._

This simple property definition directly sets and gets its data. To bind this property, we need to provide the following:

*   Methods to register(add) and remove `PropertyChangeListener`s
*   Code to fire a `PropertyChangeEvent` to the registered listeners

Because the code to perform these functions is identical for every bound property, we can delegate these functions to an instance of `PropertyChangeSupport`. A resulting Person bean containing the phoneNumber property could look as follows:

```java
package effectivevaj.bean.intro.boundproperty;

import java.beans.PropertyChangeEvent;
import java.beans.PropertyChangeListener;
import java.beans.PropertyChangeSupport;
import java.io.Serializable;

/**
	* A sample Person bean that defines a bound property
	*/

public class Person implements Serializable {
	// Create a PropertyChangeSupport instance
	//   to which we'll delegate our bound-property
	//   functionality

	private transient PropertyChangeSupport pcs =
		new PropertyChangeSupport(this);

	/** Let classes listen for property changes */
	public void addPropertyChangeListener(PropertyChangeListener l) {
		pcs.addPropertyChangeListener(l);
	}

	/** Let classes stop listening to property changes */
	public void removePropertyChangeListener(PropertyChangeListener l) {
		pcs.removePropertyChangeListener(l);
	}

	// Define a read/write/bound property named phoneNumber
	private String phoneNumber;

	/** Define property phoneNumber as readable String */
	public String getPhoneNumber() {
		return phoneNumber;
	}

	/** Define property phoneNumber as writeable String
		*  phoneNumber is bound, firing PropertyChangeEvents
		*  whenever its value changes
		*/
	public void setPhoneNumber(String phoneNumber) {
		// save the old value
		String oldNumber = this.phoneNumber;

		// set the new value
		this.phoneNumber = phoneNumber;

		// report the change
		pcs.firePropertyChange("phoneNumber", oldNumber, phoneNumber);
	}
}
```

The highlighted text in the previous example is the extra code needed to bind the `phoneNumber` property. Note that many tools, such as VisualAge for Java, can generate all of this code for you.

After binding `phoneNumber`, we can keep the telephone numbers of two people synchronized by creating listeners. For example, suppose we have a married couple, Scott and Nancy, who share a telephone number. When one spouse changes the telephone number, the telephone number should change for the other person as well.

The following code acts as glue between two `Person` beans, `scott` and `nancy`. We create event listeners (using anonymous inner classes) to listen for changes to the `phoneNumber` property of each bean. Whenever we hear that the `phoneNumber` of `scott` changes, we set the `phoneNumber` of `nancy`. We provide the same type of handling for changes to the `phoneNumber` in `nancy` as well.

```java
package effectivevaj.bean.intro.boundproperty;

import java.beans.PropertyChangeEvent;
import java.beans.PropertyChangeListener;

/**
	* Test the Person bean's bound phone number property
	*/

public class PhoneSync {

	/**
		*  Test the bound phoneNumber of the Person bean
		*  Create two people that share a phone number and
		*    create property-change listeners that will keep
		*    their phone numbers synchronized
		*/
	public static void main(String[] args) {
		// create two people who share a phone number

		final Person scott = new Person();
		final Person nancy = new Person();

		// when scott's phone changes, change nancy's
		scott.addPropertyChangeListener(
			new PropertyChangeListener() {
				public void propertyChange(PropertyChangeEvent e) {
					if (e.getPropertyName().equals("phoneNumber")) {
						nancy.setPhoneNumber(scott.getPhoneNumber());
					}
				}
			}
		);

		// when nancy's phone changes, change scott's
		nancy.addPropertyChangeListener(
			new PropertyChangeListener() {
				public void propertyChange(PropertyChangeEvent e) {
					if (e.getPropertyName().equals("phoneNumber")) {
						scott.setPhoneNumber(nancy.getPhoneNumber());
					}
				}
			}
		);

		// Run a little test...

		System.out.println("Initial phone numbers");
		System.out.println("Scott: "  + scott.getPhoneNumber());
		System.out.println("Nancy: "  + scott.getPhoneNumber());
		System.out.println();
		System.out.println("Set Scott's number to 555-1212");
		scott.setPhoneNumber("555-1212");
		System.out.println("Scott: "  + scott.getPhoneNumber());
		System.out.println("Nancy: "  + scott.getPhoneNumber());
		System.out.println();
		System.out.println("Set Nancy's number to 555-7777");
		nancy.setPhoneNumber("555-7777");
		System.out.println("Scott: "  + scott.getPhoneNumber());
		System.out.println("Nancy: "  + scott.getPhoneNumber());
	}
}
```

Creating `PropertyChangeListener`s for each `Person` bean automatically updates both `Person` beans when either telephone number changes. We will use this technique during visual composition to achieve several interesting effects, including keeping a model synchronized with a GUI. The results of running `PhoneSync` follow:

	Initial phone numbers
	Scott: null
	Nancy: null
	Set Scott's number to 555-1212
	Scott: 555-1212
	Nancy: 555-1212
	Set Nancy's number to 555-7777
	Scott: 555-7777
	Nancy: 555-7777

_Note: Some tools, such as VisualAge for Java, can generate code to perform exactly this task._

### Constrained Properties: Property Validation by Delegation

Note that this is _not_ an ideal way to do validation (you lose the invalid value...), and in practice, is rarely, if ever, used.)

Constrained properties allow registered listeners to veto a proposed change. The bean fires a `PropertyChangeEvent` before the property state is changed, and the listeners can object to the change by throwing a `PropertyVetoException`. The exception prevents the state from changing and informs the code that called the set method of the veto.

Constrained properties perform property validation through external objects. Rather than hardcoding validation inside the set method, we delegate the validation to registered listeners. Consider the benefits of this approach. If we design a text-field GUI component that has a constrained text property, we can plug in any validation we want.

In one use, we could plug in the following validation criteria:

*   All characters must be numbers.
*   There must be exactly seven characters.

In a different instance, we might plug in different validations:

*   The first character must be alphabetic.
*   The text must be a single word.
*   The text must contain at least one number.
*   The text must be greater than six characters.

Each validation process is contained in a separate `VetoableChangeListener` and registered with the text field. If any of the separate validations is not successful, the `setText()` method will fail. This is preferable to making a new subclass of the text field simply to add different validation in its `setText()` method. Think of all the subclasses that you would need to provide validation. Suppose we wanted to check that the value entered in a text field or a combo box were a date. We would need to subclass each of these classes to perform the validation. If we made their text or value properties constrained, we would only need to define the data-check logic in one validator class.

The `java.beans` package provides another concrete-implementation class, `VetoableChangeSupport`, to assist with constrained-property event processing. Constraining a property is very similar to making it bound.

The following code defines a constrained property called `phoneNumber`. Similar to our earlier bound property example, we delegate the listener tracking and event firing to another object, one of class `VetoableChangeSupport`.

```java
package effectivevaj.bean.intro.constrainedproperty;

import java.io.Serializable;
import java.beans.PropertyChangeEvent;
import java.beans.VetoableChangeListener;
import java.beans.VetoableChangeSupport;
import java.beans.PropertyVetoException;

	// A sample Person bean that defines a constrained property

public class Person implements Serializable {

	// Create a VetoableChangeSupport instance
	//   to which we'll delegate our constrained-property
	//   functionality. (Note - using lazy-instantiation this time, just 'cause I can...)
	private transient VetoableChangeSupport vcs;

	// Let classes listen for property changes
	public void addVetoableChangeListener(VetoableChangeListener l) {
		if (vcs == null)
			vcs = new VetoableChangeSupport(this);
		vcs.addVetoableChangeListener(l);
	}

	// Let classes stop listening to property changes
	public void removeVetoableChangeListener(VetoableChangeListener l) {
		if (vcs == null)
			vcs = new VetoableChangeSupport(this);
		vcs.removeVetoableChangeListener(l);
	}

	// Define a read/write/constrained property named
	//  phoneNumber
	private String phoneNumber; 

	// Define property phoneNumber as readable String
	public String getPhoneNumber() {
		return phoneNumber;
	}

	// Define property phoneNumber as writeable String
	// phoneNumber is constrained, firing 
	// PropertyChangeEvents whenever its value is
	// about to change
	public void setPhoneNumber(String phoneNumber) throws PropertyVetoException {
		if (vcs != null) {
			// report the impending change
			vcs.fireVetoableChange("phoneNumber", 
									this.phoneNumber, phoneNumber);
		}
		// change to the new value
		this.phoneNumber = phoneNumber;
	}
}
```

The highlighted text is the set of changes necessary to support a constrained property. Again, many tools, such as VisualAge for Java, can generate all of this code for you.

Registered `VetoableChangeListener`s can throw a `PropertyVetoException`, which halts the `setPhoneNumber` processing, throwing the exception to the caller of `setPhoneNumber()`.

As a simple example, we restrict a `Person`’s telephone number to exactly seven numeric characters. Our strategy to implement this is as follows:

*   Constrain the `phoneNumber` property. (We have defined that previously.)*   Create the class `StringLengthCheck` to ensure that a String is within a minimum-maximum range, as follows:
    
```java	
		package effectivevaj.bean.intro.constrainedproperty;
    
		import java.beans.PropertyVetoException;
		import java.beans.PropertyChangeEvent;
		import java.beans.VetoableChangeListener;
    
		// A VetoableChangeListener that validates that a String's
		// length falls within a given range
		public class StringLengthCheck implements VetoableChangeListener {    
			private int minimum, maximum;
    
			// constructor -- gather the min-max range for
			//					the legal string length
			public StringLengthCheck(int min, int max) {
				minimum = min;
				maximum = max;
			}

			// VetoableChangeListener notification - check to see 
			// if the length of the new string value is within 
			// the minimum/maximum range
			public void vetoableChange(PropertyChangeEvent e) throws PropertyVetoException {
				if (e.getPropertyName().equals("phoneNumber")) {
					String number = (String) e.getNewValue();
					if (number == null)
						throw new PropertyVetoException("No new value!", e);
					int len = number.length();
					if (len < minimum || len > maximum)
						throw new PropertyVetoException("Improper length: must be in range " + 
							minimum + "-" + maximum, e);
				}
			}
		}
```

*   Create class `NumberStringCheck` to ensure that all characters are numeric, as follows:

```java    
		package effectivevaj.bean.intro.constrainedproperty;
    
		import java.beans.VetoableChangeListener;
		import java.beans.PropertyChangeEvent;
		import java.beans.PropertyVetoException;
    
		// A VetoableChangeListener that validates that a 
		// String's value is completely numeric
		public class NumberStringCheck implements VetoableChangeListener {
			public void vetoableChange(PropertyChangeEvent e) throws PropertyVetoException {
				if (e.getPropertyName().equals("phoneNumber")) {
					String number = (String) e.getNewValue();
					if (number != null) {
						int len = number.length();
						for (int i = 0; i < len; i++) {
							if (!Character.isDigit(number.charAt(i))) {
								throw new PropertyVetoException(number + 
									" contains non-numeric characters", e);
							}
						}
					}
				}
			}
		}
```

*   Add instances of `StringLengthCheck` and `NumberStringCheck` to the `Person` bean as `VetoableChangeListeners`, as follows:

```java    
		package effectivevaj.bean.intro.constrainedproperty;
    
		import java.beans.PropertyVetoException;
    
		// Test our constrained phoneNumber property
		public class PhoneValidate {
			// A convenience method to handle the exceptions for us
			protected static void setPersonsPhoneNumber(Person person, String number){ 
				try {
					person.setPhoneNumber(number);
					System.out.println("Set number to " + number);
				} catch (PropertyVetoException e) {
					System.out.println(e);
				}
			}

			// Runs a simple test of our constrained phone property
			public static void main(String[] args) {
				Person scott = new Person();
				scott.addVetoableChangeListener(
				new StringLengthCheck(7, 7));
				scott.addVetoableChangeListener(new NumberStringCheck());
				// test a non-numeric phone number
				setPersonsPhoneNumber(scott,"111aaaa");
				// test a long phone number
				setPersonsPhoneNumber(scott,"5551111111111");
				// test a normal phone number
				setPersonsPhoneNumber(scott,"5551212");
				// create a new person that only checks that the
				//   phoneNumber is numeric
				Person nancy = new Person();
				nancy.addVetoableChangeListener(new NumberStringCheck());
				// test a non-numeric phone number
				setPersonsPhoneNumber(nancy,"111aaaa");
				// test a long phone number
				setPersonsPhoneNumber(nancy,"5551111111111");
			}
		}
```

As you can see from the previous test, not only can we plug in several validations, but we can also use different sets of validations for each instance of a class. The previous example will print the following to `System.out`:

Constrained test output

	java.beans.PropertyVetoException: 111aaaa contains non-numeric characters
	java.beans.PropertyVetoException: Improper length: must be in range 7-7
	Set number to 5551212
	java.beans.PropertyVetoException: 111aaaa contains non-numeric characters
	Set number to 5551111111111

Tool support for constrained properties is pretty much non-existent. Eventually, tools should support attaching validation vetoers to your beans, but we may not see this behavior for some time.

### How Can a Tool Tell if a Property is Bound or Constrained?
Tools like VisualAge for Java have two ways to determine if properties are bound and/or constrained. They can make educated assumptions, or they can find out for certain through a BeanInfo class.

If a tool sees that a bean defines method

```java
public void addPropertyChangeListener(PropertyChangeListener l)
```

it assumes that contained properties may be bound. While that is a bit over-assuming, it really does not hurt use of the bean. If the bean has any properties that are not bound, it is the bean-developer's responsibility to provide a `BeanInfo` class that explicitly states which properties are not bound.

Constrained properties are similar, but assumptions are safer. Whenever the tool sees that a property's set method throws a `PropertyVetoException`, the tool can assume the property is constrained.

Remember that properties can be both bound and constrained. In practice, most properties are only bound, and very few are only constrained.

## Event Sets

Beans may or may not fire events. If they support bound or constrained properties, they will fire events (as we have seen); they can also fire other types of events.
Event firing requires the following elements:

* An event class to transmit information about the event
* An interface that describes how the bean will notify interested parties, known as listeners
* A data structure to track listeners
* Registration methods to add and remove listeners
* Code to fire the events

For example, suppose we were creating a `WeatherStation` bean that notifies any interested parties when the sun rises or sets. We use this example throughout the following sections.

### An Event Class
The first requirement for event firing is a class to represent information about the event. Our `SunEvent` object contains the time of the event and a boolean property that tells us if the event represents the sun rising or setting.

Event classes should be immutable! Immutable objects are ones that cannot be changed. Event firing passes a reference to a single event to each listener in an unspecified order. It is important that these listeners cannot modify the event, or it could change the way processing continues for other listeners. Note that the following class is not a bean, because its superclass does not implement `java.io.Serializable`. If you like, you can define the event object as a bean but in practice this is rarely done.

Note that the event object must extend class `java.util.EventObject`. `EventObject` provides the `getSource()` method so we can determine the event origin. By extending `EventObject`, we can handle these objects generically in other methods, and some tools, like VisualAge for Java, take advantage of this when creating generic methods to forward events.

```java
package effectivevaj.bean.intro.eventsets;

import java.util.Date;
import java.util.EventObject;

// An event that represents the Sun rising or setting
public class SunEvent extends EventObject {
	private boolean risen;
	private Date date;

	public SunEvent(Object source, boolean risen, Date date) {
		super(source);
		this.risen = risen;
		this.date = date;
	}

	// return a String representation of the date
	public String getDate() {
		// return only a String representation
		//   so the user cannot modify the real date
		return date.toString();
	}

	// return whether the sun rose or set
	public boolean isRisen() {
		return risen;
	}
}
```

`EventObject` requires a `source` (a reference to the object that fired the event), which we set to our `WeatherStation` bean when firing the event. Whenever we fire events, we create an instance of the `EventObject` class and pass it to each registered listener.

### An Event-Listener Interface

Event listeners are classes that have registered interest in an event that a bean can fire. The bean notifies listener classes by calling certain methods in those listener classes. But how does the bean know which methods to call?

When defining our event, we also define a contract between the event source and its listeners. This contract specifies which methods the event source will call. Any listeners must implement these methods. Sounds like a perfect use for a Java language interface!

Therefore, we define an interface that specifies our contract. Because each listener must implement that interface, the event-source bean can determine which methods it can call. Continuing our example, we define a simple interface for our sun events.

Note that the following interface extends the `java.util.EventListener` interface. `EventListener` requires no methods. It is just a tag that indicates an interface is acting as an event listener. This assists determination of the events a bean can fire.

```java
package effectivevaj.bean.intro.eventsets;

import java.util.EventListener;

// A contract between a SunEvent source and
// listener classes
public interface SunListener extends EventListener {
	// Called whenever the sun changes position
	// in a SunEvent source object 
	public void sunMoved(SunEvent e);
}
```

We require that any `SunEvent` listeners implement a single method, `sunMoved()`. Normally, event-listener methods should take a single parameter: the event object. However, if you have a strong need, the Beans specification allows (but strongly discourages) different parameters.

### An Event-Source Bean

Now we define the source of our sun events, the `WeatherStation`. This class needs to track its listeners and fire the events. We implement this as follows, and provide a simple GUI to trigger the `SunEvent`s:

```java
package effectivevaj.bean.intro.eventsets;

import java.util.Vector;
import java.util.Date;
import java.util.Enumeration;
import java.awt.Button;
import java.awt.Frame;
import java.awt.GridLayout;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.io.Serializable;

// A sample event source - this class fires SunEvents
// to anyone watching. A simple GUI is provided
// with "rise" and "set" buttons that cause the
// SunEvents to be fired.
public class WeatherStation extends Frame implements Serializable {
	private transient Vector listeners;

	// Provide a simple GUI that triggers our SunEvents
	public WeatherStation() {
		super("Sun Watcher");
		setLayout(new GridLayout(1,0));
		Button riseButton = new Button("Rise");
		Button setButton = new Button("Set");
		add(riseButton);
		add(setButton);

		// make the "Rise" button fire "rise" SunEvents
		riseButton.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent e) {
				fireSunMoved(true);
			}
		});

		// make the "Rise" button fire "set" SunEvents
		setButton.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent e) {
				fireSunMoved(false);
			}
		});

		// Provide a means to close the application
		addWindowListener(new WindowAdapter() {
			public void windowClosing(WindowEvent e) {
				System.exit(0);
			}
		});

		pack();
	}

	// Register a listener for SunEvents
	synchronized public void addSunListener(SunListener l) {
		if (listeners == null) {
			listeners = new Vector();
		}
		listeners.addElement(l);
	}  

	// Remove a listener for SunEvents
	synchronized public void removeSunListener(SunListener l) {
		if (listeners == null) {
			listeners = new Vector();
		}
		listeners.removeElement(l);
	}

	// Fire a SunEvent to all registered listeners
	protected void fireSunMoved(boolean rose) {
		// if we have no listeners, do nothing...
		if (listeners != null && !listeners.isEmpty()) {
			// create the event object to send
			SunEvent event = new SunEvent(this, rose, new Date());

			// make a copy of the listener list in case
			//   anyone adds/removes listeners
			Vector targets;
			synchronized (this) {
				targets = (Vector) listeners.clone();
			}

			// walk through the listener list and
			//   call the sunMoved method in each
			Enumeration e = targets.elements();
			while (e.hasMoreElements()) {
				SunListener l = (SunListener) e.nextElement();
				l.sunMoved(event);
			}
		}
	}
}
```

Our event source can now notify interested listeners. The `synchronized` keywords and cloning are necessary to avoid possible problems if the list of listeners changes while we are notifying listeners. Pressing the Rise and Set buttons fires an appropriate `SunEvent` to any registered listeners.

Typically, developers will create a `fire` method similar to the previous `fireSunMoved()` method, but the method is not required. The event-firing code can appear in any method you prefer. VisualAge for Java will create a method similar to this fire method, taking a SunEvent as its argument.

### A Sample Event Listener

In our example, the weather channel informs any interested parties about the sun rising or setting. Continuing this example, Mrs. Jones assigns her class the task of graphing the behavior of the sun. They must watch a weather channel to find out at exactly what time the sun rose and set each day, logging this information in their notebook. We can model this example using a Student class as follows:

```java
// A sample SunListener, logging when the sun rises and sets
public class Student implements SunListener {
	private String name;

	public Student(String name) {
		this.name = name;
	}

	public void sunMoved(SunEvent e) {
		log(name + "\tlogs : " + (e.isRisen() ? "rose" : "set") +
				" at " + e.getDate()); 
	}

	protected void log(String text) {
		System.out.println(text);
	}
}
```

Note that `Student` implements `SunListener`, defining the details of a `sunMoved()` method. Any number of `Student`s may watch the weather channel to hear the time of sunrise and sunset.

```java
package effectivevaj.bean.intro.eventsets;

// A simple test of the SunEvent source and listeners
public class WeatherTest {
	// Run a test on the weather station, using Scott's
	//  kids as sample students
	public static void main(String[] args) {
		// create a new sun event source
		WeatherStation w = new WeatherStation();

		// add some students to listen for sun rise/set
		w.addSunListener(new Student("Nicole"));
		w.addSunListener(new Student("Alex"));
		w.addSunListener(new Student("Trevor"));
		w.addSunListener(new Student("Claire"));

		// display the GUI for the weather channel
		w.setVisible(true);
	}
}
```

Running WeatherTest and pressing the Rise and Set buttons results in something like the following:

	Nicole logs : rose at Thu Apr 29 14:55:21 PDT 1999
	Alex   logs : rose at Thu Apr 29 14:55:21 PDT 1999
	Trevor logs : rose at Thu Apr 29 14:55:21 PDT 1999
	Claire logs : rose at Thu Apr 29 14:55:21 PDT 1999
	Nicole logs : set at Thu Apr 29 14:55:22 PDT 1999
	Alex   logs : set at Thu Apr 29 14:55:22 PDT 1999
	Trevor logs : set at Thu Apr 29 14:55:22 PDT 1999
	Claire logs : set at Thu Apr 29 14:55:22 PDT 1999

Warning!

The above listener methods run in a specific order each time an event is fired. This is because we stored the listeners in an ordered data structure. **_HOWEVER, There is NO guarantee of event-notification order in the Beans specification!_** Unless a bean documents its event-firing behavior as ordered, you should not _assume_ any particular order of notification. Beans could store their listeners in any data, and they may report them in a different order every time they fire an event.

Also, note that event firing is **synchronous**. That is to say, the event source calls only one event handler at a time. Most beans fire their events in this manner (although there is nothing to stop you from writing a bean that fires events asynchronously). Because of this, you should perform only **short** operations in your event handlers. Think of it this way: The next listener has to wait for you, so you should be courteous and return as quickly as possible. If you need more complex operations, spawn another thread from your event handler.

### Determining Which Events Are Fired

Events are useful when writing code by hand, but they require a lot of code. Fortunately, bean-builder tools can provide assistance in generating this code. But how can a bean-builder tool determine which events can be fired so it can assist?

Recall that methods define properties using certain naming conventions; we define events in a similar manner. Bean-builder tools look for methods with the following naming pattern:

```java
public void addSomeName(SomeName l)
public void removeSomeName(SomeName l)
```

When a tool sees a method that matches this naming pattern, it checks to see if `SomeName` is an interface that implements `EventListener`. (Remember that tag we added to the `SunListener` interface?) A method matching this pattern defines an event set.

Once identified, the bean-builder tool examines the event listener interface to see which methods it requires. These methods are possible events that the bean can fire. Using this information, the bean-builder tool can tell the user which events a bean can fire, enabling the user to associate an event from one bean with an action on another. We will see later how the bean-builder tool can use this information to help a user generate an application.

## Methods

The last type of bean features are methods. You may have guessed by now that bean methods are leftovers; that is, they are any public methods that remain after we determine properties and event sets. Bean-builder tools can present a list of methods to the programmer, stating available actions for any given bean. Bean tools do not treat method features as anything special. Normally you just connect to them as the result of some event being fired.

## Bean-Builder Tools and Beans

Now that we know how tools can recognize bean features, we can explore how these tools use beans.

Bean-builder tools typically have a tool bar or palette that holds pictures of commonly used beans. The user selects one of these beans and drops it in a design area that represents the class being built. The user can drop multiple beans in the design area; some tools allow only graphical user-interface beans, while other tools allow any beans.

The user can ask for a property sheet for any bean in the design area. Property sheets are dialogs that contain a list of property names and values. The bean-builder tool determines which properties are available and then calls the appropriate get method to display the current values of those properties. The user can then modify writeable properties (properties with set methods).

Values set in a property sheet represent the initial state of that particular bean instance. Of course, the program can modify the property values during execution by invoking their set methods. When a bean-builder tool generates source code for a bean, it creates new statements for each bean, as well as calls to each set method to set the property values specified in the property sheet.

Most bean-builder tools enable the user to specify interactions between beans. These interactions start with one bean firing an event, and end by calling a method or setting a property in another (or the same) bean. Some tools provide wizards to specify this interaction, while others enable the user to draw lines between components, choosing the source event and target feature via pop-up menus. The bean-builder tool can generate an event listener to respond to these events and call the appropriate target method. VisualAge for Java provides the most advanced bean-builder tool available, its Visual Composition Editor.

### Introspection and the Role of BeanInfo Classes

Introspection is the process of determining which features a bean supports. In its most basic form, introspection is simply a process of using the Java core reflection classes to determine which public methods the bean defines and match those methods against the bean naming patterns.

The introspection process examines the bean class and proceeds to each superclass. The process inspects public methods in each class, adding them to the list of features. If any duplicate features exist, subclass definitions take precedence. Often this provides the desired functionality; sometimes it can fall short.

How can we hide features from the user of a bean-builder tool? How can we allow only expert users to see some features, while non-expert users see only very basic features? How can we change the way the user interacts with the properties in a property sheet, or enable several properties to be set as a group? Explicit metadata is required to address these and other issues, describing exactly what we want. The bean specification calls this metadata BeanInfo. BeanInfo are classes that are delivered along with beans. Any bean may have a BeanInfo class, which describes in detail the features provided by that bean.

BeanInfo classes are associated with a bean by a simple naming convention. The name of the BeanInfo class is the name of the bean with "BeanInfo" as its suffix. For example, a bean named Person could have a PersonBeanInfo class that describes it. BeanInfo classes all implement the java.beans.BeanInfo interface, which requires methods that return arrays of PropertyDescriptors, MethodDescriptors, and EventSetDescriptors as well as general information about the bean itself.

If the introspection process discovers a BeanInfo class, it queries that BeanInfo class to determine features instead of examining public method names. The FeatureDescriptors (PropertyDescriptors, MethodDescriptors, and EventSetDescriptors) provide the names and descriptions of the features. Each FeatureDescriptor can contain additional information, stating if a feature should be hidden or considered expert. The bean-builder tool uses the FeatureDescriptor information to filter the features that are available. PropertyDescriptors can also specify a property editor, one means of customizing the way a user edits a property’s value in the bean-builder tool.

Finally, the BeanInfo class provides a BeanDescriptor, which describes the overall bean, possibly including a Customizer (which provides customized editing for the entire bean) and icons to represent the bean in the bean-builder tool’s palette. BeanInfo classes are tedious to write, but fortunately, many tools, such as VisualAge for Java, generate them for you.

Keep in mind that once you define a BeanInfo class, it is in charge of describing bean features. If you add new methods to your bean, bean builder tools will not know about them unless you add the features to the BeanInfo class as well. 
