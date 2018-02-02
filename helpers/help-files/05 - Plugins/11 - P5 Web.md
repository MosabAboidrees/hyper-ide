
## p5.web, Ajax widgets and more

p5.web is the Ajax web widget GUI library for Phosphorus Five and Hyperlambda, in addition to that it takes care
of everything related to the web in P5. This is the part that makes it possible for you to use Active Events
such as **[create-widget]** and **[set-widget-property]**. In addition, it contains helper Active Events which
allows you to modify stuff such as the response HTTP headers, access the session object, and even entirely
take control over the returned response, through events such as **[p5.web.echo]** and **[p5.web.echo-file]**.
To a large extent, it wraps p5.ajax, allowing you to use Active Events to create, delete and manipulate
Ajax widgets.

**Notice** - p5.web is built around ASP.NET WebForms, but it is also the _only part of P5_ which is referencing `System.Web`.
This implies that you can very much use Phosphorus Five in for instance a Microsoft MVC application, and probably
also in .Net Core if you want to - Or for that matter a desktop application, or a mobile app with Xamarin Forms.
If you choose to do this, then p5.web is the only project you cannot use. This makes Phosphorus Five also highly
useful for development in .Net Core or Microsoft MVC, since it allows you to use for instance Hyperlambda as a
_"scripting language"_ for your Microsoft MVC applications. So although Phosphorus Five is arguably created
around WebForms, 99% of it can still be used in any other CLR type of application, due to its highly modular
architecture.

### Creating your first Ajax web widget

Below is a piece of code that allows you to create an Ajax web widget, which handles the _"onclick"_ event on
the server, and modifies your widget's text property when clicked.

```hyperlambda
create-widget:my-widget
  element:h3
  innerValue:Click me!
  onclick
    p5.types.date.now
    set-widget-property:my-widget
      innerValue:x:/@p5.types.date.now?value.string
```

The above Hyperlambda, will create an H3 HTML element, having the value of _"Click me!"_, which when clicked,
will change its value to the server's date and time. The **[onclick]** callback to the server, is of course
wrapped in an Ajax HTTP request automatically for you. Allowing you to focus on your domain problems, and not
on the Ajax internals. There exists three basic Active Events for creating such mentioned Ajax widgets in P5.
They are for the most parts almost identical, except for some minor details which sets them apart.

### The common arguments to all create widget events

These arguments are common to all create widgets events, meaning **[create-literal-widget]**,
**[create-container-widget]** and **[create-void-widget]**. Notice, if you use the generic **[create-widget]**
event, your widget type will be automatically determined, according to whether or not you supply an
**[innerValue]**, a **[widgets]** collection, or none of the previously mentioned arguments.

The most important argument is the value of your create widget invocation node. This argument becomes the ID
of your widget, and how you will reference it, both on the server through Hyperlambda, and on the client through
JavaScript. The ID argument is optional, and if you do not supply it, then an automatically generated ID will
be assigned your widget.

* __[parent]__ - Defines the parent widget for your widget. Mutually exclusive with __[after]__ and __[before]__
* __[position]__ - Defines the position in the parent list of widgets, only applicable if you define __[parent]__
* __[before]__ - An ID to a widget from where your widget should be injected before. Mutually exclusive with __[parent]__ and __[after]__
* __[after]__ - An ID to a widget from where your widget should be injected after. Mutually exclusive with __[parent]__ and __[before]__

Notice, you can only declare one of **[parent]**, **[before]** or **[after]**, and only if you declare a
**[parent]**, you can declare a **[position]**. By cleverly using the above arguments, you can insert your Ajax
widgets, exactly where you wish in your page's DOM structure, and p5.web will automatically persist your Ajax
widgets across postbacks and Ajax requests. In addition to the positional arguments, all widgets also optionally
takes some other common arguments. These are listed below.

* __[visible]__ - Boolean, defines if the widget is initially rendered as visible or invisible
* __[element]__ - Defines which HTML element to render your widget with
* __[events]__ - List of widget lambda Active Events, that are coupled with your widget, and only alive as long as your widget exist

### HTML attributes and widget events

In addition to these especially handled arguments, you can in addition add up any argument you wish. Depending
upon the name of your argument, it will either be handled as an HTML attribute to your widget, or an event of
some sort. If your argument starts with the text _"on"_, it will be assumed to be some sort of client side event -
Either a DOM JavaScript event if your node has a value, or a server-side Ajax event if it has no value, but
contains a collection of children nodes of itself. Below is an example of how to create a widget with a class
attribute and a style attribute for instance.

```hyperlambda
create-widget:some-other-widget
  element:h3
  style:"background-color:LightBlue;"
  class:some-css-class
  innerValue:Colors
```

Since neither of our custom arguments above (the style and class arguments) starts out with _"on"_, they are
treated as custom HTML attributes, and not as events. If you want to create an Ajax event instead, you could
do this like the following code illustrates.

```hyperlambda
create-widget
  element:h3
  innerValue:Colors, hover your mouse over me!
  onmouseover
    set-widget-property:x:/../*/_event?value
      style:"background-color:LightGreen"
```

Since our above argument starts out with _"on"_, P5 automatically creates an Ajax server-side event, evaluating
the associated lambda every time this event is raised. Notice one detail in the above lambda, which is that it
does not declare an explicit ID. This means that the widget will have an automatically assigned ID, looking
something like the following; `"x3833968"`. This means that we do not know the ID of our widget from inside
our **[onmouseover]** event. However, this is not a problem for us, since the ID of the widget will be forwarded
into all Ajax events, and lambda events automatically. Each time an event is raised, it will have an **[\_event]**
argument dynamically passed into it, which is the server-side and client-side ID of our widget. By referencing
this **[\_event]** argument in our **[set-widget-property]** expression above, we are able to modify the widget's
properties.

In fact, if you have a list of widgets, automatically created, inside for instance a loop, which creates rows
and cells for a table for instance - Then you _should not_ give these widgets an explicit ID, but rather rely
upon the automatically generated ID, to avoid the problem of having multiple widgets on your page, with the
same ID - Which would be a severe logical error. If you use your own explicit IDs, you should also take great
care making sure they are unique for your page. Which means that you would probably end up creating some sort
of namespacing logic for things that are used on multiple pages, and injected as reusable controls into your
page - Which is a common pattern when using P5.

### [create-widgets]

There also exists a plural overload Active Event for creating widgets, which at first sight might not seem
that useful. However, every now and then, it fits just perfect. Its name is **[create-widgets]**, and it just
takes a bunch of widgets, such as **[literal]**, **[void]**, **[container]**, or your own extension widgets.
The **[create-widgets]** overload allows you to create multiple widgets in one go, each having a separate
**[parent]**, etc. But more importanly, it allows you to create extension widgets, without being forced to
create a wrapper widget, wrapping your extension widget. Every now and then, this ability is crucial, to keep
your DOM tidy, and not having to create redundant widgets, simply to get to the widget you wish to create
immediately. So even though its name might be misleading, since you would expect to be using it to create
_multiple_ widgets - It actually allows you to create _fewer_ widgets sometimes, than what you'd have to
create if you were using the default **[create-widget]**, or one of its overloads. Below is an example.

```hyperlambda
create-widgets
  micro.widgets.modal:modal-widget
    class:micro-modal
    widgets
      h3
        innerValue:Modal widget
      p
        innerValue:Foo bar
      div
        class:right
        widgets
          button
            innerValue:Close
            onclick
              delete-widget:modal-widget
```

### JavaScript and client-side DOM events

If you create an argument that starts out with the text _"on"_, and have a value, instead of children nodes,
you can put any arbitrary JavaScript you wish into this value. This would inject your JavaScript into the
attribute of your widget, as rendered on the client-side, allowing you to create JavaScript hooks for DOM
HTML events. Imagine something like this for instance.

```hyperlambda
create-widget
  element:h3
  position:0
  innerValue:JavaScript events - Hover your mouse over me!
  onmouseover:"alert ('foo');"
```

The above lambda, will render an HTML element for you, which when the mouse hovers over it, creates an alert
JavaScript message box. Any JavaScript you can legally put into an _"onmouseover"_ attribute of your HTML
elements, you can also legally put into the above value of **[onmouseover]**. Using this logic, you could
completely circumvent the server-side Ajax parts of Phosphorus Five, and roll your own logic - While still
retaining the ability to use all the other features of the library.

### In-visible properties and events

Sometimes you have some value, or event for that matter, which you want to associate with your widget, but
you don't want to render it back to the client, but instead only access it from your server. Or as would be
the case for in-visible events, not render them as your widget's HTML, but be able to access them through the
JavaScript API of P5. This is easily done, by simply prepending an underscore (\_) or (.) in front of your widget's
attribute or event. This would ensure that this attribute or event is not rendered as a part of your markup,
but only accessible on the server (for attributes), or through the JavaScript API (for events). For instance,
to create a value, which you can only access on the server, you could do something like this.

```hyperlambda
create-widget
  element:h3
  innerValue:Click me to see the server-side value of [.foo]
  .foo:foo value
  onclick
    get-widget-property:x:/../*/_event?value
      .foo
    set-widget-property:x:/../*/_event?value
      innerValue:Value of [.foo] is '{0}'
        :x:/@get-widget-property/*/*?value
```

Notice how the **[get-widget-property]** retrieves the **[.foo]** value, while the **[set-widget-property]** sets
the **[innerValue]** of the widget to a string formatted value, where the `{0}` parts becomes the value retrieved
from the widget's **[.foo]**'s value. Notice also how this value is only acessible from the server, and not
visible or possible to retrieve on the client. Neither by inspecting the DOM, HTML or by using JavaScript.
By starting an attribute with an underscore (\_) or a (.), it is completely invisible for the client in every way.
If you prepend an event with underscore (\_), or a period (.) - The results are similar. Consider this code.

```hyperlambda
create-widget:some-invisible-event
  element:h3
  innerValue:Click me to invoke [.foo]
  .onfoo
    set-widget-property:x:/../*/_event?value
      innerValue:[.onfoo] was raised
  onclick:@"p5.$('some-invisible-event').raise('.onfoo');"
```

Notice that if you inspect the DOM or HTML of the above output, the **[.onfoo]** widget event is completely
invisible in all regards. Only when you attempt to raise it, you realize it's there. The above code also uses
some parts of P5's JavaScript API in its **[onclick]** value, which is documented in p5.ajax - In case you're
interested in the details. For the record, almost all arguments to your widgets are optional.

### Container widgets

So far we have only used the **[create-literal-widget]**, indirectly though. The literal widget, can only
contain text or HTML, through its **[innerValue]** property. However, a container widget, allows you to create
an entire hierarchy of widgets. Instead of having an **[innerValue]** property, it contains a collection of
children widgets, through its **[widgets]** property. Let's illustrate with an example.

```hyperlambda
create-widget
  element:ul
  widgets
    literal
      element:li
      innerValue:Element no 1
    literal
      element:li
      innerValue:Element no 2
```

Notice, we still use the same **[create-widget]** event, but this time we supply a **[widgets]** collection,
instead of an **[innerValue]** property. This ensure that we'll end up with a **[container]** widget instead
of a **[literal]** widget. Another thing to notice in the above code, is that the children nodes of the
**[widgets]** property of our container widget, can have the following 3 values (actually 4 values, but our
4th widget is special, and explained later).

* __[literal]__ - Declares a literal widget
* __[container]__ - Declares an inner container widget
* __[void]__ - Declares a void widget

These 3 values maps to the **[create-literal-widget]**, **[create-container-widget]** and **[create-void-widget]** -
And you could, if you wanted to, create a single widget at the time, and then fill it manually with its children
widgets, by using one of the create Active Events. However, you will probably find the above syntax more easy and
simple to use, for creating rich hierarchies of widgets, where all your widgets are known during the initial
creation. And of course, each widget above could also have its own set of events and attributes, both hidden
and visible. An example is given below.

```hyperlambda
create-widget
  element:ul
  style:"font-size:large"
  widgets
    literal
      element:li
      innerValue:Element no 1
      style:"background-color:LightGreen;"
      onclick
        micro.windows.info:Widget 1 was clicked
    literal
      element:li
      innerValue:Element no 2
      style:"background-color:LightBlue;"
      onclick
        micro.windows.info:Widget 2 was clicked
```

You can also of course create container widgets that have container widgets as their children widgets, as deeply
as you wish.

### Void widgets

The void widget is useful for HTML elements that have neither content nor children widgets. Some examples are the
HTML _"br"_ element, _"input"_ elements, and _"hr"_ elements. These HTML elements are characterized by that they
completely lack content. They are often used to create HTML form elements, such as is given an example of below.


```hyperlambda
create-widget:some-void-widget
  element:input
  style:"width:400px;"
  placeholder:Type something into me, and click the button ...
  class:form-control
create-widget
  element:input
  type:button
  value:Click me!
  class:btn btn-default
  onclick
    get-widget-property:some-void-widget
      value
    micro.windows.info:You typed; '{0}'
      :x:/@get-widget-property/*/*?value
```

In the above example, we create two form elements; One textbox and one button. Both are created using the void
widget type, since they have neither any **[innerValue]**, nor any **[widgets]** collection - Hence, they
implicitly become void widgets. Above you also see an example of how you can add arbitrary attributes, that
affects the rendering of your widgets in some regards. The example above for instance, is using the **[type]**
and **[placeholder]** attributes from HTML5, in addition to the **[value]** attribute for our button. To use
the p5.web project in Phosphorus Five, requires some basic knowledge about HTML, and preferably HTML5 - In
addition to some basic knowledge about CSS (optionally, if you wish to style your HTML).

### Ninja tricks for declaring HTML elements of your widgets

When you create a container widget, you can optionally declare its children widgets' HTML element as their name.
This saves you a couple of lines of code for each widget. If you do, the type of widget (container, literal, void),
will be implicitly figured out according to whether or not you've added an **[innerValue]** argument, **[widgets]**
collection, or none of the previously mentioned. For instance, to create a container widget, that has one _"p"_
element child, you could do something like the following.

```hyperlambda
create-widget
  widgets
    p
      innerValue:Some text
```

The widget is still a fully fledged widget, and you can add event handlers, lambda events, and so on to it -
Just like as if it was declared like the following.

```hyperlambda
create-widget
  widgets
    literal
      element:p
      innerValue:Some text
```

However, you save one line of code, and your lambda widget declarations becomes more easily understood, since
they semantically reflect the HTML hierarchy they create more directly.

### The default HTML elements for widgets

By default, all create widget Active Events, will use the following HTML elements for rendering the widgets on
the client.

* __[create-container-widget]__ - "div"
* __[create-literal-widget]__ - "p"
* __[create-void-widget]__ - "input"

This means you can omit the **[element]** arguments if you are creating the above HTML elements.

### Creating custom reusable widgets

To create a custom widget, all you have to do, is to create an Active Event, containing a period (.).
Then make sure this Active Event returns exactly _one_ valid widget for a **[widgets]** argument to a container
widget. At this point you can use your Active Event as an extension widget.

```hyperlambda
// This Active Event becomes a "custom widget type"
create-event:foo.bar
  return
    literal
      innerValue:Howdy world

create-widget
  widgets

    // At this point, we can use our "custom widget" as if it was any other widget type
    foo.bar:some-id-to-widget
```

The above lambda, first declares an Active Event with the name of **[foo.bar]**. This Active Event returns one
**[literal]** widget. Then we use our **[foo.bar]** as if it was just another widget type, in our **[widgets]**
collection of our root container widget, created using **[create-widget]**.

The above Ninja trick, allows you to create reusable complex widgets as Active Events, which you can include in
any other parts of your solutions - Just as if they were normal widgets. By creating rich hierarchies of widgets,
through Active Events as above, you can create any amount of complexity in your extension widgets, such as
TreeViews, DataGrids, etc. Your Active Event extension widgets can also take arguments, allowing for you to
parametrize your widgets during creation. It is important that such custom widget Active Events, returns _one_
and _exactly one_ widget back to the caller. This single widget however, can be a **[container]** widget,
containing several children widgets of itself. Notice, the single return value from your Active Event, will
have the widget id passed in as value to your invocation, automatically becoming the id for the return widget
from your Active Event. If no ID is given, an automatic ID will be generated, just like for any other widgets.

### The [text] widget

There exist a fourth widget, although it is not technically a widget. It is simply the ability to inject text
into your resulting HTML at some specific position of your DOM. This can be useful for instance when you wish
to inject inline JavaScript or CSS into your resulting HTML for some reasons. This widget has no properties or
attributes, and cannot be de-referenced in any ways - Neither in JavaScript nor on the server side. Imagine
the following.

```hyperlambda
create-widget
  widgets
    text:@"<style type=""text/css"">
.my-class {
    background-color:rgba(255,255,0,.8);
    font-size:large;
    font-family:Comic Sans MS
}</style>"
    literal
      class:my-class
      innerValue:This should be rendered with yellow background and some 'funny' font
```

Notice that this is not actually a widget, but simply a piece of text, rendered into the resulting HTML,
at the position you declare it. This means, that it cannot be de-referenced in any ways on the server side,
or the client side - And the only way to update it, is to remove it entirely, deleting its parent widget.
In general terms, it is considered advanced, due to the previously mentioned reasons - And you should not use it,
unless you are 100% certain about that you understand its implications. Besides, using inline JavaScript or CSS is
anyways considered an _"anti-pattern"_ when composing your DOM/HTML. And doing just that, is one of the few
actual use-cases for it. However, it's there for those cases when a normal widget simply won't cut it for you.

### [get-widget-property] and [set-widget-property]

These two Active Events, allows you to retrieve and change any property you wish. The serialization is 100%
automatic back to the client, and all the nitty and gritty details are taken care of for you automatically.
We have alread used both of them already, but in this section, we will further dissect them, to gain a complete
picture of how they work. The **[get-widget-property]** allows you to get as many properties as you wish in one
go. You declare each property you wish to retrieve as the name of children nodes of the invocation, and p5.web
automatically fills out the values accordingly. Consider this.

```hyperlambda
create-widget
  element:h3
  innerValue:Foo
  class:bar
  onclick
    get-widget-property:x:/../*/_event?value
      innerValue
      class
    create-widget
      element:pre
      innerValue:x:/..
```

Our above **[create-widget]** invocation, inside of our **[onclick]** Ajax event handler, is a _"Ninja Trick"_ to
see the lambda tree, after evaluation of some piece of Hyperlambda. Below is an example of of how this might look
like after evaluation.

```hyperlambda
onclick
  _event:xa886f5b
  get-widget-property
    xa886f5b
      innerValue:Foo
      class:bar
    create-widget:x09aefa78
      element:pre
      innerValue:x:/..
```

As you can see above, the **[get-widget-property]** returns the ID for your widget, and beneath the ID, it
returns each property requested as a _"key"_/_"value"_ child node. Another interesting fact you see above,
is that we use a root iterator expression - Still it displays only the **[onclick]** node. This is because
during the evaluation of your **[onclick]** event, the onclick node _is_ your root node. This is similar to
how it works when you invoke a lambda object, or dynamically created Active Event, using for instance **[eval]**.
The rest of your lambda object, is not accessible at this point. You can also retrieve multiple widget
properties in one invocation, by supplying an expression leading to multiple IDs. Consider this.

```hyperlambda
create-widget
  element:h3
  widgets
    literal:first-literal
      innerValue:Some value
      class:bar1
      element:span
    literal:second-literal
      innerValue:Click me!
      class:bar2
      element:span
      onclick
        _widgets
          first-literal
          second-literal
        get-widget-property:x:/../*/_widgets/*?name
          innerValue
          class
      create-widget
        element:pre
        innerValue:x:/..
```

The **[set-widget-property]** works similarly, except of course, instead of retrieving the properties,
it changes them. Consider this.

```hyperlambda
create-widget
  element:h3
  widgets
    literal:first-literal
      innerValue:Some value
      class:bar1
      element:span
    literal:second-literal
      innerValue:Click me!
      class:bar2
      element:span
      onclick
        _widgets
          first-literal
          second-literal
        set-widget-property:x:/@_widgets/*?name
          innerValue:Your second widget was clicked
          class:bar-after-update
```

Normally, you will only update a single widget's properties though. But for those rare occassions, where you
for instance wish to update the CSS class of multiple widgets in one go, being able to do such in a single go,
is a nifty feature. Notice, you can use **[set-widget-property]** to set a property which does not previously
exist on your widget. This will simply add the property. You can also use the **[delete-widget-property]**
event to entirely delete a property or attribute from your widget. A widget with a null value in a property,
still actually has that property. Its value is simply null. This is necessary to be able to render HTML5
attributes such as _"controls"_ or _"disabled"_ on an element. Hence, you cannot supply a null value to remove
a widget's attribute using **[set-widget-property]**. You can also change a widget's visibility using the
**[set-widget-property]** Active Event. But these types of properties cannot be removed. Only attributes
rendered to the client as HTML attributes can be removed. You _cannot_ change a widget's **[parent]**,
**[after]**, **[before]**, or **[position]** after it has been created.

Hint!
There is also an event called **[p5.web.widgets.property.list]**, which returns a list of _all_ properties,
for one or more specified widget(s). This Active Event can either be given an expression, leading to multiple
IDs, or a constant, showing the properties of only one widget at the time.

### Retrieving an entire hierarchy of widget properties

Sometimes, you wish to retrieve every single widget property, recursively, from some specified root widget.
For these occassions, you have the **[p5.web.widgets.properties.get]** Active Event (plural form). This Active
Event will return all properties of the requested name(s), below one or more specified root widget. To see all
form element values for instance, in your form, in one go, you could use something like this for instance.

```hyperlambda
p5.web.widgets.properties.get:cnt
  value
```

The above is actually a _"Ninja trick"_ to serialize all form elements from some specified parent widget
recursively.

### Changing and retrieving a widget's Ajax events dynamically

The same way you can update and change properties of widgets, you can also change their lambda object for what
occurs when some Ajax event is raised. To do this, you would use the **[p5.web.widgets.ajax-events.get]** and
the **[p5.web.widgets.ajax-events.set]** Active Events. Consider this code.

```hyperlambda
create-widget
  innerValue:Click me!
  onclick
    set-widget-property:x:/../*/_event?value
      innerValue:I was clicked, click me once more! It tickles!
    p5.web.widgets.ajax-events.set:x:/../*/_event?value
      onclick
        set-widget-property:x:/../*/_event?value
          innerValue:Thank you for tickling me!
```

The above code dynamically changes the lambda that is evaluated as the widget is clicked when you click it the
first time. The **[p5.web.widgets.ajax-events.get]** returns its existing lambda object. Try out the following.

```hyperlambda
create-widget
  innerValue:Click me!
  onclick
    p5.web.widgets.ajax-events.get:x:/../*/_event?value
      onclick
    create-widget
      element:pre
      innerValue:x:/..
```

**Notice** - You can also explicitly raise a widget's Ajax events from your own lambda objects, by invoking
**[p5.web.widgets.ajax-events.raise]**. To raise the **[onclick]** event on a single widget for instance,
you could do something such as the following.

```hyperlambda
create-widget:foo-bar
  innerValue:Click me!
  onclick
    set-widget-property:x:/../*/_event?value
      innerValue:I was clicked programmatically!
p5.web.widgets.ajax-events.raise:foo-bar
  onclick
```

Phosphorus will not discriminate in any ways between events dynamically raised by the user clicking some
object himself, or an event raised by your lambda in the manner shown above - Except it won't (obviously) have
to do an explicit client/server Ajax roundtrip.

### Lambda events for widgets

A widget can also have dynamically declared Active Events associated with it. These are Active Events which
are accessible, for any context within your page - But only as long as the widget is existing on your page.
This feature allows you to declare dynamically created Active Events, which only exists, as long as your widget
exist. Which again makes it easier for your separate parts of your page, to communicate with other parts of
your page, and create a more _"object oriented feel"_ to your widgets. Consider this code.

```hyperlambda
create-widget
  events
    my-namespace.foo-event
      micro.windows.info:Foo event raised
  widgets
    literal
      innerValue:Click me!
      onclick
        my-namespace.foo-event
```

Such lamba events are functioning identically to any other dynamically created Active Events, or lambda objects,
and can take arguments, and return values and nodes to the caller. They're simply Active Events, which only
lives as long as your widget lives, and only for your page. Also these lambda events can be dynamically changed,
the same way you could with Ajax events, using the **[p5.web.widgets.lambda-events.set]**, and the
**[p5.web.widgets.lambda-events.get]**. Both of these Active Events, functions similarly to their Ajax events
counterparts.

**Notice** - You can also use **[p5.web.widgets.lambda-events.list]** and **[p5.web.widgets.ajax-events.list]**
to inspect which lambda and/or Ajax events are declared for a specific widget.

### [oninit], initializing your widgets

The **[oninit]** is a special type of event, which is only raised every time your widget is made visible. It
is useful for initialization and such of your widget, but besides from its semantics, it works similarly to an
Ajax event. Though none of its internals, not even its existence, is ever rendered back to the client in any ways.
It can be useful for populating a _"select"_ HTML widget, with _"option"_ items, created dynamically from
data in your database, for instance. But there are many other useful scenarios for this event. Semantically,
it behaves similar to an Ajax event, but it cannot be raised from the client. It renders no additional data
back to the client. When **[oninit]** fires, all other events and properties have already been created for your
widget. This allows you to invoke widget lambda events associated with your widget, and retrieve their properties,
from within your **[oninit]** lambda object, after your widget has been created - But before it is rendered back
to the client. Below is an example of **[oninit]** in action.

```hyperlambda
create-widget
  innerValue:This text will never show up on your page!
  oninit
    set-widget-property:x:/../*/_event?value
      innerValue:Dynamically changed property during [oninit]
      element:h3
```

### Deleting widgets and clearing widget collections

In addition to the above mentioned events, you also have the **[delete-widget]** and **[clear-widget]**. The
first one entirely deletes a widget from your page, while the second one empties its children collection.

### Retrieving widgets

There are also several helper Active Events to retrieve widgets, according to some criteria. These are listed below.

* __[p5.web.widgets.get-parent]__ - Returns the parent widget(s) of the specified widget(s)
* __[p5.web.widgets.get-children]__ - Returns the children widgets of the specified widget(s)
* __[p5.web.widgets.find]__ - Returns the widgets that have the properties listed, with optionally, the values listed
* __[p5.web.widgets.find-like]__ - Same as above, but doesn't require an exact match, only that the widget's properties contains your value(s)
* __[p5.web.widgets.find-ancestor]__ - Returns all ancestor widgets with the specifed properties and values
* __[p5.web.widgets.find-ancestor-like]__ - Same as above, but is happy as long as the value contains the value(s) specified
* __[p5.web.widgets.find-first-ancestor]__ - Returns first ancestor widget with the specifed properties and values
* __[p5.web.widgets.find-first-ancestor-like]__ - Same as above, but is happy as long as the value contains the value(s) specified
* __[p5.web.widgets.list]__ - List all widgets that have IDs matching the specified string(s)
* __[p5.web.widgets.list-like]__ - Same as above, but is happy as long as the ID(s) contains the value(s) specified
* __[p5.web.widgets.exists]__ - Yields true for each widget that exists matching the specified ID(s). This event has a __[widget-exists]__ alias.

Almost all the above mentioned events, returns the same structure back to caller, which looks like the following;

```hyperlambda
some-widget-retrieval-event
  bar1
    container:foo
  bar2
    container:foo
```

The first level of children, are the currently iterated criteria passedin as **[\_arg]**. Then comes one or more
children nodes, having a name defining which type of widget this is, and a value being the ID of our widget.
Have that in mind as we look at these Active Events and how they work. The exception to the above structure,
is **[p5.web.widget.exists]**, which simply returns a true/false value if the widget(s) exists.

#### [p5.web.widgets.get-parent]

Returns the parent widget of one or more specified widget(s). Example given below.

```hyperlambda
create-widget:foo
  widgets
    literal:bar1
      innerValue:bar1
    literal:bar2
      innerValue:bar2
p5.web.widgets.get-parent:x:/../*/create-widget/**/literal?value
```

The above code, uses an expression leading to each value of each **[literal]** node within the **[widgets]** part
of the **[create-widget]** invocation. This means that it will return the parent widget for both the _"bar1"_ widget
and the _"bar2"_ widget, which happens to be the same. The output should look something like this, showing the
type of widget as the name, and the ID of the parent widget as its value.

```hyperlambda
/* ... rest of code ... */
p5.web.widgets.get-parent
  bar1
    container:foo
  bar2
    container:foo
```

Notice how **[p5.web.widgets.get-parent]** optionally can take an expression as its argument. This means we have
to return an additional _"layer"_ before returning the actual parent widget(s), since we need to return parent
widget to which widget to the caller. If we had simply returned the parent as the first node of
**[p5.web.widgets.get-parent]**, we would not know who this was a parent to, since we can supply multiple widgets
as arguments. This is a general pattern for all of our widget retrieval events.

#### [p5.web.widgets.get-children]

Returns the children widgets of one or more specified widget(s). Example below.

```hyperlambda
create-widget:foo
  widgets
    literal:bar1
      innerValue:bar1
    literal:bar2
      innerValue:bar2
p5.web.widgets.get-children:foo
```

The above code uses a constant, instead of an expression as our previous example - And should return something
like the following.

```hyperlambda
/* ... rest of code ... */
p5.web.widgets.get-children
  foo
    literal:bar1
    literal:bar2
```

Even though we do _not_ use an expression in the above example, we still return the children widgets in a similar
structure, as we did with our previous example of **[p5.web.widgets.get-parent]**, which used an expression. This
is to make sure we always return data to the caller in the same format. If we had used an expression leading to
multiple source widgets, instead of the constant of _"foo"_, we would have had multiple return nodes beneath
**[p5.web.widgets.get-children]** in our result.

#### [p5.web.widgets.find] and [p5.web.widgets.find-like]

These events takes (optionally) the root widget from where to start your search, and require you to parametrize
them with children nodes, having at least a name, and optionally a value. The name of the node, is some attribute
that must exist on your widget, for it be returned as a match. The value of the attribute to look for, is an
optionally value for that attribute, which the widget must either have an exact match of (**[p5.web.widgets.find]**),
or contain (for the **[p5.web.widgets.find-like]** event).

Let's see an example.

```hyperlambda
create-widget:foo
  widgets
    literal:bar1
      innerValue:bar1
      class:foo-class
    literal:bar2
      innerValue:bar2
p5.web.widgets.find:foo
  innerValue:bar1
  class:foo-class
p5.web.widgets.find:foo
  innerValue:bar
p5.web.widgets.find-like:foo
  innerValue:bar
```

The above code has two invocations to **[p5.web.widgets.find]**, and one to **[p5.web.widgets.find-like]**.
The first invocation, requires an exact match for the **[innerValue]**, and the **[class]** attributes,
being respectively _"bar1"_, and _"foo-class"_. This invocation will return only the _"bar1"_ widget.
The second invocation won't yield any matches, since it requires an exact match, and no widgets have the
innerValue of exactly _"bar"_, since they both have numbers behind their _"bar"_ string parts. The third
invocation will yield both widgets as its return value, since they both have an **[innerValue]** which
_contains_ the text _"bar"_. The result will look something like this.

```hyperlambda
/* ... rest of code ... */
p5.web.widgets.find
  foo
    literal:bar1
p5.web.widgets.find
p5.web.widgets.find-like
  foo
    literal:bar1
    literal:bar2
```

These Active Events can also take expressions as their arguments.

```hyperlambda
create-widget:foo1
  widgets
    literal:bar1
      innerValue:bar1
      class:foo-class
    literal:bar2
      innerValue:bar2
create-widget:foo2
  widgets
    literal:bar3
      innerValue:bar1
      class:foo-class
    literal:bar4
      innerValue:bar2
p5.web.widgets.find:x:/../*/[0,2]?value
  innerValue:bar1
```

Which of course result in this result.

```hyperlambda
/* ... rest of code ... */
p5.web.widgets.find
  foo1
    literal:bar1
  foo2
    literal:bar3
```

#### [p5.web.widgets.find-ancestor] and [p5.web.widgets.find-ancestor-like]

These two Active Events are similar to the **[p5.web.widgets.find]** events, except of course, instead of
searching downwards in the hierarchy from the root widget supplied - They search upwards in the ancestor chain
for a match, from the currently iterated widget. These Active Events requires (for obvious reasons) the caller
to actually supply a value, and will not tolerate a default value as our previously mentioned widget retrieval
Active Events did. Example code given below.

```hyperlambda
create-widget:foo1
  class:foo-class-1
  widgets
    container:bar1
      widgets
        literal:starting-widget-1
          innerValue:Foo bar
create-widget:foo2
  class:foo-class-2
  widgets
    container:bar2
      widgets
        literal:starting-widget-2
          innerValue:Foo bar
p5.web.widgets.find-ancestor-like:x:/../**/literal?value
  class:foo-class
```

Which of course will yield the following result.

```hyperlambda
p5.web.widgets.find-ancestor-like
  starting-widget-1
    container:foo1
  starting-widget-2
    container:foo2
```

#### [p5.web.widgets.list] and [p5.web.widgets.list-like]

These Active Events simply lists all widgets, with an ID matching some (optional) filter supplied. Notice, if
you do not supply a filter, this Active Event will yield every single widget in your page, starting from the
root _"cnt"_ widget, declared in your Default.aspx markup. Example code is given below.

```hyperlambda
create-widget:test-foo1
  widgets
    literal:test-bar1
      innerValue:Bar 1
    literal:test-bar2
      innerValue:Bar 2

// Will only return "test-bar2"
p5.web.widgets.list:test-bar2

// Will not return anything
p5.web.widgets.list:test

// Will return all three widgets
p5.web.widgets.list-like:test

// Will return only the two literals
_lit
  test-bar1
  test-bar2
p5.web.widgets.list:x:/-/*?name
```

The above code will yield something like this.

```hyperlambda
/* ... rest of code ... */
p5.web.widgets.list
  literal:test-bar2
p5.web.widgets.list
p5.web.widgets.list-like
  container:test-foo1
  literal:test-bar1
  literal:test-bar2
_lit
  test-bar1
  test-bar2
p5.web.widgets.list
  literal:test-bar1
  literal:test-bar2
```

One point though, as in many of the other widget retrieval Active Events, is the fact that you are also getting
the type of widget returned, as the name of the node - While you get the ID of the widget, returned as the value.
Notice though that **[p5.web.widgets.list]** does not return an injected node for the widget you start out your
search from - Simply since there are no such node(s) or criteria given to it. In addition, the same widget might
match several criteria, making it impossible to put into one specific filter. This means it yields one level less
than all other widget retrieval Active Events.

#### [p5.web.widgets.exists]

This Active Event allows you to check for the existence of one or more specified widget(s). It can take either a
constant, or an expression. Below is an example.

```hyperlambda
create-widget:test-foo1
  widgets
    literal:test-bar1
      innerValue:Bar 1
    literal:test-bar2
      innerValue:Bar 2
p5.web.widgets.exists:test-bar2
```

Which will yield the following result.

```hyperlambda
/* ... rest of code ... */
p5.web.widgets.exists
  test-bar2:bool:true
```

### Storing stuff and state in the web "context"

In ASP.NET you have lots of different storage objects which you can use, such as the Session object, Application
object, Cache, Cookies, and so on. These same storage facilitates are mapped into Phosphorus Five, through the
C# Active Events you can find in the storage folder of this project. They all more or less obey by the same rules
(Active Event syntax), which allows you to easily change between any one of them, later in your project, as you
see fit. This gives you a flexible environment for development, where you can easily move your data storage around,
from different objects, as you see fit later.

Below is a complete list of all of these Active Events.

* __[p5.web.session.set]__ - Has private (.) alias
* __[p5.web.session.get]__ - Has private (.) alias
* __[p5.web.session.list]__ - Has private (.) alias
* __[p5.web.application.set]__ - Has private (.) alias
* __[p5.web.application.get]__ - Has private (.) alias
* __[p5.web.application.list]__ - Has private (.) alias
* __[p5.web.context.set]__ - Has private (.) alias
* __[p5.web.context.get]__ - Has private (.) alias
* __[p5.web.context.list]__ - Has private (.) alias
* __[p5.web.cache.set]__ - Has private (.) alias
* __[p5.web.cache.get]__ - Has private (.) alias
* __[p5.web.cache.list]__ - Has private (.) alias
* __[p5.web.cookie.set]__ - Has private (.) alias
* __[p5.web.cookie.get]__ - Has private (.) alias
* __[p5.web.cookie.list]__ - Has private (.) alias
* __[p5.web.header.set]__
* __[p5.web.header.get]__
* __[p5.web.header.list]__
* __[p5.web.params.get]__
* __[p5.web.params.list]__

Not all of them can be interchanged with each other, since for instance things like HTTP headers, cannot tolerate
complete node structures and more complex objects - In addition to that storing stuff you'd normally store in
your session, obviously does not at all make sense storing in an HTTP header. But all of the above events,
obeys by the same set of rules, and returns more or less similarly structured results. Most of the above Active
Events also have native alias versions, which allows you to invoke them with a "." in front of their name. If you do,
then you are allowed to retrieve, list, and modify protected data - Which is really just data starting with either
an underscore (\_) or a period (.). This ensures that you can store things into these objects, which is only
accessible through C#. Which is an additional security feature, for things you do not wish some random piece of
Hyperlambda to access. Notice, there are also three Active Events defined in the p5.webapp project, which allows
you to store _"page values"_ (ViewState).

* __[p5.web.viewstate.set]__ - Has private (.) alias
* __[p5.web.viewstate.get]__ - Has private (.) alias
* __[p5.web.viewstate.list]__ - Has private (.) alias

The above three mentioned events, also obeys by the same structure as the once listed above.

### Accessing the ASP.NET Session object

If we start out with the _"Session"_ object from ASP.NET, there are three basic Active Events which allows you to
use your session.

* __[p5.web.session.set]__
* __[p5.web.session.get]__
* __[p5.web.session.list]__

For those not aquinted with how the session object works, it is normally a memory based in-process storage,
for objects you wish to associate with a single session (user), for as long as a user is actively using your
web site. When the server is rebooted, the user leaves your website, or some other disaster occurs, all session
variables are discarded and lost. The session object _can_ (optionally, through configuring your web.config for
instance), be stuffed into a database, making it cross-process enabled, and so on. But by default, it is in memory.
All objects stored into the session, are only accessible for the currently visiting client, and only for the
lifespan of his session. Often it times out after some minutes (20 minutes by default) of inactivity, which means
the user will loose all his session variables, if he does not somehow, interact with your website for 20 minutes.
To show you an example of it, imagine the following code.

```hyperlambda
p5.web.session.set:test.my-session-variable
  src:Some piece of data goes here
p5.web.session.get:test.my-session-variable
```

In the above code, we first set a session variable, name it _"test.my-session-variable"_, and give it the static
value of _"Some piece of data goes here"_ - Before we retrieve it again. The retrieval does not have to be in the
same request though. As long as your session has not timed out, you can refresh your browser, and evaluate the
following code, and your original value will still be returned back to you.

```hyperlambda
p5.web.session.get:test.my-session-variable
```

All of the object storage Active Events in p5.web, takes a source argument, the same way **[add]** and **[set]**
does. Which allows you to create fairly complex sources, using Active Event sources for instance. To store a
slightly more complex object into your session, you could use something like the following.

```hyperlambda
p5.web.session.set:test.my-session-variable
  src
    my-node:My value
      my-other-node
        some-integer:int:5
p5.web.session.get:test.my-session-variable
```

Above we see an example of adding typed objects into the session, which of course is no problem. Like with **[set]**
though, you can only have _one_ source. This means that if you have complex hierarchies of nodes, you have to make
sure you have one root node. This is because you're setting _one item_ in your session. If you wish to put arrays
into your session, you'll have to have them as children of a single root node. The following code will raise an
exception.

```hyperlambda
// Throws an exception!!
p5.web.session.set:test.my-session-variable
  src
    my-node:My value
    my-other-node
```

#### Listing your session keys using [p5.web.session.list]

Sometimes it can be useful to have a list of which session keys you have in your current session object. This is
easily done using **[p5.web.session.list]**. An example of usage is shown below.

```hyperlambda
p5.web.session.list
```

If you provide a value, or an expression as its value, this will be used as a filter that your keys must match,
in order to be returned. Example is given below.

```hyperlambda
p5.web.session.list:test.my-session-variable
```

The above would yield, assuming you've still got your session objects from our first example in your session,
the following result.

```hyperlambda
p5.web.session.list
  test.my-session-variable
```

To retrieve all values, from all session objects, could easily be done combining this invocation with
a **[p5.web.session.get]** invocation.

```hyperlambda
p5.web.session.list
p5.web.session.get:x:/-/*?name
```

### Accessing the application object

The application object has an API which is 100% identical to the session object, and consists of these three
Active Events.

* __[p5.web.application.get]__
* __[p5.web.application.set]__
* __[p5.web.application.list]__

The only difference is the underlaying implementation, which stores your values in the global application object
instead of your session object. The application object is global for all users of your website. Besides from that,
it is really quite similar to the session object, and only lives for the life time of your server process,
being stored in memory, the same way as your session values. To see some examples of it in use, see the session
examples, and simply replace _"session"_ with _"application"_.

### Accessing your context object

The HttpContext object is also identical to both the session and the application object, except of course,
it stores things in the HttpContext object instead. To understand the difference, check out the documentation
for the HttpContext class in .Net Framework. However, basically, it stores values only for the current request
to your server. Replace the _"session"_ parts with _"context"_ to use the HttpContext object instead in our
above examples.

### Accessing your cache object

Also this object is identical to both your session object, application object, and context object - Except it
uses the keyword _"cache"_. It carries one additional (optional) argument though, which is **[minutes]**, which
defined for how many minutes the cache object you insert should be valid for. This argument is only applicable
when invoking **[p5.web.cache.set]**, and has no relevance for any of the other cache Active Events.
**[minutes]** defaults to 30 unless explicitly set to something else.

### Cookies in P5

The cookies Active Events are similar to the session Active Events, except you cannot retrieve a cookie that
you set in the same response. This is because when you set a cookie, you are modifying the HTTP response object,
while when you retrieve a cookie, you are retrieving it from the HTTP request. Hence after setting a new cookie
value, then it won't be accessible before you return the response back to the client, and the client
makes another request to your server. In addition, type information is partially lost when you set a cookie.
To illustrate with an example, run these two piece of code.

```hyperlambda
p5.web.cookie.set:some-cookie
  src
    foo:bar
      some-integer:int:5
```

Then run this code.

```hyperlambda
p5.web.cookie.get:some-cookie
```

As you can see in your last piece of code, your cookie value is returned as a string. It is quite easily
converted back into a lambda object though, by running it through **[hyper2lambda]** with the code below.
However, since the cookie collection is a simple string storage, while the session and application object in
ASP.NET can contain any arbitrary objects, there is no way for us to intelligently store type information
in the cookie collection, without bloating the HTTP requests/responses - Hence, we simply avoid adding type
information.

```hyperlambda
p5.web.cookie.get:some-cookie
hyper2lambda:x:/-/*?value
```

This is because although you can store _"objects"_ in your session, application and cache objects, etc - The
cookie collection of your browser, can only handle strings. However, your node(s) are converted correctly
into Hyperlambda before stored into your cookie collection, which allows you to convert it easily back to
lambda using **[hyper2lambda]**.

### HTTP headers

Now you could if you wanted to, store lambda code in your HTTP headers. However, doing such a thing, probably
makes no sense what so ever. HTTP headers are for informing the browser, and/or potential proxies, about some
state of your application. And using it as a general storage, would be like using a crocodile to hunt raindeers.
However, setting an HTTP header obeys by the same API as all other storage events. To have your web app return
a custom header to your client, you could do something like this.

```hyperlambda
p5.web.header.set:foo
  src:bar
```

If you make sure you inspect your HTTP request, before you evaluate the code above, you will see that the
return value from your server, contains one additional HTTP header called _"foo"_, having the value of _"bar"_.
HTTP headers get and set operations are sufffering from the same problem as cookies for the record, which is
that _"get"_ retrieves its data from the request, while _"set"_ puts its data into the response - Which implies
that stuff you put into an HTTP header using a set operation, is not accessible for a consecutive get operation.

### HTTP GET parameters

To access your HTTP GET parameters, you can use **[p5.web.params.get]** or **[p5.web.params.list]**. Since the
GET parameter of a request, is a read-only type of collection, there exists no set. Try to add up the following
string at the end of your URL; `&foo-key=bar-value`. Then run the following code.

```hyperlambda
p5.web.params.get:foo-key
```

Now of course, since modifying the HTTP GET parameter collection of a request, requires changing the URL of a
request, there exists no _"set"_. If you wish to _"set"_ an HTTP GET parameter, you'll need to change the URL
by doing a redirect of the client somehow. In addition to retrieving the GET parameters of your HTTP request,
you can also retrieve POST parameters, using the following Active Events.

* __[p5.web.post.get]__
* __[p5.web.post.list]__

There are also a generic parameter collection, with the following events.

* __[p5.web.params.get]__
* __[p5.web.params.list]__

The latter, will retrieve all parameters from within your context. This will include web server settings, etc,
and is hence probably not that useful.

### Raw request and response

If you wish, you can completely bypass the default HTTP serialization and deserialization, and instead take
complete control of every aspect of both the rendering, and the parsing, of both an HTTP request, and a response.
This is useful if you are creating web services, and/or returning files to the caller for instance. For such
cases, you have the **[p5.web.echo]** and **[p5.web.echo-file]** Active Events for creating your own response.
And you have the **[p5.web.request.get-body]**, and the **[p5.web.request.get-method]** events. These Active
Events allows you to access the _"raw"_ HTTP request, as sent by the client - And create your own response,
exactly as you see fit. The default HTTP request/response model in P5, uses POST requests for each Ajax request,
and returns JSON to the caller. However, if you wish, you can completely bypass this model, and create your own.
Imagine if you wish to return a file to the caller for instance. This is easily achieved by creating a page,
which contains logic like this.

```hyperlambda
p5.web.echo-file:~/documents/private/README.md
```

If you evaluate the above code, it will throw a JavaScript exception, since the Ajax method invoked when
you click the _"Evaluate"_ button expects the server to return JSON. However, if you create a page, which
during initial loading, instead of creating an Ajax web widget hierarchy, evaluates the above lambda - Then
you will download the above file to the client when you load that page.

To see an example, create a **[page]** page in for instance Hypereval with the URL of "/download-my-file".
Then change its code to the following.

```hyperlambda
p5.web.echo-file:~/documents/private/README.md
p5.web.header.set:Content-Type
  src:text/plain
```

Then save your page and click _"Preview"_, and you should see your file. If you change the code for your page
to the following.

```hyperlambda
p5.web.echo:@"This is foo calling!"
p5.web.header.set:Content-Type
  src:text/plain
```

You will see some statically created text instead. Notice, once you invoke **[p5.web.echo]** or
**[p5.web.echo-file]**, then you cannot invoke it again. This is because the implementation in .Net actually
_"kills"_ the current thread. This means that if you want to dynamically build up your content, you have to
build it first, and have **[p5.web.echo]** be the last piece of logic in your page. The **[p5.web.echo-file]**
event will also throw an exception if the currently logged in user is not authorized to reading the file you
supply to it. Both **[p5.web.echo]** and **[p5.web.echo-file]** can optionally take expressions, leading to
either one or more pieces of text, or one file. Creating web services, using **[p5.web.echo]**, which instead
of returning HTML to the client, or returns some other piece of data, is quite easy using this technique.

**Notice** - You can also return Hyperlambda to the client using the **[p5.web.echo]** event, since it
will automatically convert whatever expression it is given to text, which allows you to return some sub-set
of your tree to the caller.

### Getting the raw HTTP request

You can also retrieve the raw HTTP request, by using the **[p5.web.request.get-body]** Active Event. This
Active Event will return the raw HTTP request sent by the client, giving you complete access to it, to do
whatever you wish with it. This is a quite useful feature of P5, since it allows you to create web service
end-points, where you can for instance pass in Hyperlambda, or some other piece of machine readable type of
request, which is intended to be used by machines and web services, instead of browser clients. If you wish
to directly save the request, without first putting it into memory, you can save some memory and CPU cycles
by directly saving the request body using the **[p5.web.request.save-body]** Active Event, which takes a
constant or expression leading to a filename on your server. Notice, this event
requires the currently logged in user context to be able to write to the path supplied. The latter event
is quite useful if you are implementing some sort of REST web service, which is supposed to _"PUT"_ some file
for instance. However, when dealing with more complex types of HTTP requests, there also exists helper
Active Events in the p5.mime project for things such as parsing more complex MIME requests.

### Additional request helper events

You can also get the HTTP method of your request, using the **[p5.web.request.get-method]** Active Event,
in addition to that you can have P5 make its best guess of whether or not the request originated from a mobile
device using the **[p5.web.request.is-mobile]** Active Event. The latter is not 100% perfect, since a mobile
device is not required to identify itself as such to your server - But it is good enough for most cases,
and will do a decent job, determining if the client is some sort of mobile device or not.

### Modifying your HTTP status code and text

You can modify the status message and the status code of your response using these two Active Events.

* __[p5.web.response.set-status-code]__ - Sets the status code, requires an integer, or an expression leading to an integer as its input
* __[p5.web.response.set-status-description]__ - Sets the status message, tolerates anything that is convertible into text

### Creating Web Services, lambda style

One thing you should realize about **[p5.web.echo]** and **[p5.web.request.get-body]**, is that you can both pass
in, and return Hyperlambda, which you then convert to lambda, for then to evaluate it as such. This feature of P5,
allows you to pass code from your client, to a server, and have the server evaluate your code, for then to return
code back again, which the client evaluates on its side. This allows the client to decide what code is to be
evaluated on your web-server endpoints, which at least in theory, makes it possible for you to create one single
web service point, for all your web-service needs. In addition, it lets you massage the output from some web
service endpoint, before it is returned, which can possibly reduce your response, and network traffic
significantly compared to a specialized web service endpoint, which possibly yields values back to the client,
that the client is not interested in. This feature has some serious security issues, which you should consider
before you go down this path though. But if you are 100% certain of that you trust the client that initiated
the request (due to being a part of your own intranet for instance), then you can safely use this feature, and allow
your clients to invoke code in your web service endpoints.

Notice, to reduce the risks associated with this approach, please read up on the **[eval-whitelist]** Active Event,
which allows you to declare a sub-set of legal Active Events to your **[eval]** invocation.
If you combine this feature, with the PGP cryptographic features of P5 from p5.mime, requiring having your web
service invocations cryptographically signed, before you evaluate them as Hyperlambda - You can create an
additional layer of protection, further safe-guarding you against malicious requests. This way you have a
cryptographically secure context, giving your guarantees of that the invocation towards your web service,
was created by some client, which you trust 100% - Since the signing process of an invocation, makes sure it
has not beeen tampered with in any ways after leaving the client. And only the client owning the private key that was
used to sign the invocation, can create a signature matching your expectations. Combined with **[eval-whitelist]**,
this approach should allow you to create a rock-solid secure lambda web service end-point.

To use this feature, you would have to pass in your invocations as MIME messages, using the p5.mime library of P5.
This would also allow you to encrypt your invocations, making it impossible for an adversary, to listen in on the
conversation, between your client(s) and your server(s). If you encrypt your web service invocations, you can even
securely use features such as the Active Event **[login]**, in your embedded Hyperlambda, to change the Application
Context user ticket for your web service invocations, giving you further rights, having your invocation being
evaluated within an explicit user context.

### Additional helper events

In addition to the above mentioned Active Event, you also have the following events, which are fairly self-explaining in nature.

* __[p5.web.request.is-ajax-callback]__ - Returns true if this is an Ajax request
* __[p5.web.send-javascript]__ - Send a piece of JavaScript to the client
* __[p5.web.include-javascript]__ - Includes a piece of JavaScript snippet persistently on page
* __[p5.web.include-javascript-file]__ - Includes a JavaScript file persistently on the client
* __[p5.web.include-css-file]__ - Includes a CSS file persistently on the client
* __[p5.web.set-location]__ - Redirects the client to the specified location
* __[p5.web.get-location]__ - Returns the current location (URL)
* __[p5.web.get-location-url]__ - Returns only the main URL, without any HTTP query parameters
* __[p5.web.get-root-location]__ - Returns the root URL of your web application
* __[p5.web.get-relative-location]__ - Returns the relative location
* __[p5.web.get-relative-location-url]__ - Returns the relative location, without any HTTP query parameters
* __[p5.web.reload-location]__ - Refreshes the client
* __[p5.web.return-response-object]__ - Returns your own object as JSON back to caller. Requires usage of the JavaScript API of p5.ajax