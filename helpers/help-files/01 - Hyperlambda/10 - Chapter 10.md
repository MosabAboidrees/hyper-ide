# Active Events preludium

In this chapter, we will start discussing the heart of P5; *Active Events*. So far, we have used this name many times during the book, but we have never formalized it, or explained it in any ways. After having read this chapter, you will have a 10.000 feet astronaut's overview of the answer to the question; *"Why Active Events?"* In the next chapter, we will discuss the technical details of Active Events.

For the record, I once wrote an MSDN article about Active Events, which you can find [here](https://msdn.microsoft.com/en-us/magazine/mt795187). If you're reading this book in some sort of dead forrest distribution model, you can find the article here; https://msdn.microsoft.com/en-us/magazine/mt795187

## Definition

Active Events are first and foremost a design pattern. It is however a design pattern, that largely replaces most other design patterns in this world. Simply because it solves the same problems as dozens of other design patterns do combined. Active Events also happens to solve these problems in a very beautiful way, making sure you are able to create better cohesion in your software, while retaining a larger degree of encapsulation, and end up with increased means to apply polymorphism. Hence, it also largely becomes a replacement to OOP.

An Active Event is an alternative way of invoking functionality. It is, in such a way, a substitute to what traditional programming refers to as *"functions"* and *"methods"*. Among its defining traits, is the fact that every Active Event, can accept the same set of arguments. In fact, they all have the same input signatures, which can be condensed down to *"a bunch of nodes"* or *"a lambda object"* to be specific. They also all return the exact same stuff, which again, you guessed it, is *"a lambda object"*.

### Terms

**Definition 0**; *"OOP"* is an acronym, and it means *"Object Oriented Programming"*. Its main design, is based upon the assumption, that things in software, can be *"classified"*, and *"types"* can be created, that encapsulates everything related to some part of your application.

**Definition 1**; *"Polymorphism"* is the art of substituting one piece of functionality with another piece of functionality. Preferably such that any old functionality, is not broken as a consequence of applying the new functionality. Polymorphism is about having old code, invoke new functionality, without breaking.

**Definition 2**; *"Encapsulation"* is the art of being able to hide the internals of your components. This is crucial to be able to create large scale software systems, without forcing the software engineer to understand every single detail of the system as a whole. It is arguably a prerequisite for creating systems, where more than one developer is contributing to the system.

**Definition 3**; *"Cohesion"* is the art of making a piece of logic become *"self sufficient"*, and not dependent upon some other prerequisite. For instance, if I say; *"New years eve is booring. But the fireworks was nice this year"* - Then the last sentence cannot be isolatet, without loosing a crucial parts of its context. If you drop the first sentence, the reader will not know if you're talking about new years eve, or 4th of July. Hence, the last sentence, is arguably said to have *"bad cohesion"*.

### The importance of good software design

All of the above, are concepts you would want to increasingly take advantage of, as you create more and more complex systems. A software system, lacking any of the above, is often referred to as being *"badly designed"*. A software architect, and developer, should strive to *"increase cohesion"*, *"apply encapsulation"*, and *"facilitate for polymorphism"*. If he or she does not do this, the result often becomes *"spaghettic code"*.

Things in your software should preferably be *"substitutable with other things"* (polymorphism), *"hide their internals from the rest of the system"* (encapsulation) and *"create value by themselves without dependencies and pre-requisites"* (cohesion). If they do, you have created reusable software, easily maintained and understood by others - And the system is said to have a good architecture and design. In traditional programming, a software system with good architecture, is often said to be _"SOLID"_. Software guys (and gals) loves acronyms.

## Active Events' architecture

Active Events, just so happens to drastically increase all of the above parameters. Or at the very least, make it much easier for you, to accomplish the above. Which often comes as a surprise to software developers, used to other programming languages - Since they are often taught that the above concepts, can only be achieved using OOP. Hyperlambda *have no OOP mechanisms*. Hence, arguably, it *refactors* our best practices, and instigates the creation of an entirely new way of thinking about software architecture. While paradoxically not loosing anything, since you can easily combine Active Events and Hyperlambda with OOP, and *traditional* programming. You could for instance create an Active Event in C#, where you use as many of the traditional OOP mechanisms as you see fit. To illustrate this point, realize that Hyperlambda and P5 is entirely built in C#, _with_ OOP mechanisms.

One example of how OOP breaks down cohesion, is how OOP for instance, during the creation of objects, forces you to decorate these objects. Already at this point, you have lost cohesion. Another example is how OOP often requires you to pass in an object to a method, which requires you to instantiate this input object, at which point you've also lost cohesion. A third example, is how a method in an OOP language has a signature, which means you've already lost the ability to substitute your method invocation, with most other methods in your system, and you have lost a lot of your ability to apply polymorphism.

By completely dropping the concept of OOP, we have very effectively *eliminated* an entire dimension of potential problems related to encapsulation, polymorphism, and cohesion.

The perfect object, according to the rules of encapsulation, has no public methods, properties, or fields. In addition, it can be substituted with any other object in your system, facilitating for 100% perfect polymorphism. This just so happens to largely define lambda objects.

### How Active Events solves these problems

Any Active Event can accept any input arguments, intended for any other Active Event. This implies that you can, at least in theory, substitute any Active Event, with any other Active Event. In addition, the invocation to one Active Event, is a simple string, which can be changed as you see fit. Hence, Active Events are said to facilitate for *100% polymorphism*, where anything can be substituted with anything.

Since there are no types in Hyperlambda, we have eliminated a whole dimension of cohesion problems. When you have no types, there is no *"external magical state"*, which increase dependency between your code, and some other piece of code. Without types, it is meaningless to talk about initializing, or decorating your objects, *obviously*! - Hence, good cohesion becomes an *implicit side-effect* of Active Events. Breaking the rules of cohesion, is almost impossible with Active Events.

Since no lambda objects contains any public methods, properties or fields, beyond the ones they all contain, necessary to traverse and change the object - There are no semantic differences between two different lambda objects. Arguably hence, the lambda object becomes perfectly encapsulated - And exchanging any lambda object with any other, is actually quite simple.

### The LSP problem

In classic OOP, an example of something that is extremely unintuitive, is the (L)iskov (S)ubstitution (P)rinciple, in regards to for instance squares and reactangles. In OOP, you are according to LSP never supposed to inherit a *"Square class"* from a *"Rectangle class"*. Neiter can you do the opposite. The reasoning for that, is beyond the scope of this book, but can easily be found with a Google search. This is beacuse in OOP a rectangle is not a subset of a square, neither is the opposite true. This creates unintuitive structures for you in your code, where your code doesn't match the mental model you *"intuitive feel"* is the right model for creating your class hierarchies.

The LSP problem often in practice, goes so deep in fact, that any software system which has a *"good architecture"*, often ends up not being able to utilise inheritance at all. This results in that polymorphism becomes impossible, without breaking the rules of SOLID code. A symptom of this problem, can be seen by how all great software architects will repeat to you; *"Prefer composition, don't use inheritance"*. So basically, the entire axiom that OOP was built around, which is classes inheriting from other classes, becomes more or less by the definition of *"good software architecture"*, impossible to use, and arguably obsolete, without risking ending up with *"spaghetti code"*, violating LSP.

In Hyperlambda, the Liskov Substitution Principle makes absolutely no sense, and the entire problem's axiom, becomes the equivalent of asking the question; *"Is you lunch married?"*. In Hyperlambda, a vehicle can be a cat, a soccer field, a piece of metal - _Without_ breaking LSP. And you can easily marry any lambda object with any other lambda object!

> Im Hyperlambda, Unicorns and Pegasuses truly do exist!

This makes it possible for you, to apply whatever *"object model you feel for applying"*, to your code, making the LSP problem completely *obsolete*.

## The proof is in the pudding

One of the things that have historically bothered me the most about ASP.NET, and more specifically Web Forms, is that the *"Page"* class inherits from the *"Control"* class. You can see this clearly for yourselves, if you create a new .ASPX page, and follow its inheritance chain upwards in its hierarchy. This is clearly a violation of the Liskov Substitution Principle, and arguably a *very bad thing to do*. I am certain of that the developers who originally architected this class hierarchy, had very good reasons to do this, and I don't mean to attack them in any ways here - But it is still a very real problem, creating a lot of potential problems for developers.

> Now if Microsoft couldn't get this right, spending 5 years creating .Net Framework in the first place, what makes you think you can get it right?

In Hyperlambda the above problem makes absolutely no sense at all, and dynamically loading a page from your System42 CMS' database, and modifying it to become a *"user control"*, often makes perfect sense, and can be easily done, with a handful of lines of code.

Below is a piece of Hyperlambda, which if you evaluate in your *"Apps/Executor"* will do two things for you.

* Create a new CMS page
* Create an Active Event that becomes a *"user control"* event

Evaluate this piece of Hyperlambda first, in your *"Executor"*.

```
/*
 * This inserts a new page into our database.
 */
p5.data.insert
  p5.page:/sys42-examples-dox-page
    role:root
    name:A page, that becomes a control
    template:/system42/apps/CMS/page-templates/no-navbar-menu.hl
    type:lambda
    lambda:@"create-widget
  element:button
  innerValue:Click me!
  onclick
    set-widget-property:x:/../*/_event?value
      innerValue:I was clicked!"

/*
 * This creates a "user control" Active Event,
 * that can take a [page] argument, that transforms
 * an existing [p5.page] object, into a "user control".
 */
create-event:sys42.examples.dox.create-control-from-page

  /*
   * Selects the given [page] argument from database, 
   * assuming it is a [p5.page] object type.
   */
  select-data:x:@"/*/*/p5.page/""={0}"""
    :x:/../*/page?value
  if:x:/@select-data/*
    not
    throw:Sorry, couldn't find that page

  /*
   * Adding results from above [select-data]
   * into [return] at the end of event.
   * Making sure we transform its Hyperlambda content
   * to a lambda object first.
   * Notice, we also "sanity check" page, making 
   * sure it has only one root node, that actually creates
   * some widget first!
   */
  hyper2lambda:x:/@select-data/*/*/lambda?value
  if:x:/@hyper2lambda/*?count
    !=:int:1
    or:x:/@hyper2lambda/0?name
      !~:create
    throw:Sorry, I cannot transform that page to a control
  add:x:/../*/return
    src:x:/@hyper2lambda/*

  /*
   * Modifying the [create-widget] invocation to either a
   * [literal], [void] or [container] node, according to what
   * type the root node of our page is.
   */
  if:x:/../*/return/0?name
    ~:-container-
    or:x:/../*/return/0/*/widgets

    /*
     * Page creates a "root container" widget.
     */
    set:x:/../*/return/0?name
      src:container

  else-if:x:/../*/return/0?name
    ~:-literal-
    or:x:/../*/return/0/*/innerValue

    /*
     * Page creates a "root literal" widget.
     */
    set:x:/../*/return/0?name
      src:literal

  else

    /*
     * Page creates a "root void" widget.
     */
    set:x:/../*/return/0?name
      src:void

  return
```

Then open your CMS and create a new *"lambda"* page, and paste in the following code.

```
create-widget
  parent:content
  widgets
    literal
      element:h1
      innerValue:A page that consumes a page!
    sys42.examples.dox.create-control-from-page
      page:/sys42-examples-dox-page
```

At this point, you have created a new lambda page, which contains a *"user control"*, which you have declared as an Active Event. This *"user control"* takes a **[page]** argument, being the URL of your page, which it will dynamically load from your database, and transform into either a **[void]**, **[literal]** or **[container]** widget - Depending upon what type of root widget your page is actually creating.

Hence, you have created a *"user control"*, that can dynamically load any page from your CMS, as long as the page obeys by a couple of simple rules, and transform this page into a reusable *"user control"*, which you can inject into any other parts of your page.

The only sane way to do this in most other programming languages, would be to create an iframe, or something similar. Using P5 and Hyperlambda, we were able to do this, such that we had a single resulting page, with the markup perfectly preserved as a single HTML page, in a 100% perfectly generic way, allowing you to use almost any *"page"* object as a *"control"* - **WITHOUT** even having to worry about the Liskov Substitution Principle.

Hence, we have arguably proven, that *"anything can be interchanged with anything"* using Hyperlambda, and that the Liskov Substitution Principle, arguably makes no sense in Hyperlambda at all, and has been rendered in its entirety as an *obsolete thing*!

With that in mind, let's move on to the next chapter, where we will look at the internals of Active Events, and start creating some for ourselves.

[Chapter 11, Active Events](chapter-11.md)
