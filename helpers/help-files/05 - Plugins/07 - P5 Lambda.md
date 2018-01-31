
## p5.lambda, "keywords" in Phosphorus Five

This is the core of the _"non-programming language keywords"_ of Phosphorus Five - Although, programming language,
doesn't quite fit the description, since there is no programming language per se, but only an execution tree.
However, for all practical concerns, it feels like a programming language. It allows you to create code
in Hyperlambda, that will be evaluated as computing instructions. It is also Turing complete in regards
to execution, and allows you to do everything you can do in any other programming languages.

### [eval], the heart of p5.lambda

The main Active Event in p5.lambda, and probably most important event in P5, is definitely **[eval]**.
With **[eval]**, you can evaluate any Node hierarchy, such as illustrated below.

```hyperlambda
_x
  foo1:hello
  foo2:there
  foo3:stranger
  foo4:do you wish to become my friend?
for-each:x:/@_x/*?value
  create-widget
    innerValue:x:/@_dp?value
```

If you evaluate the above code, you will see that it creates 4 widgets for you. This is because the
above Hyperlambda is first converted to a lambda structure, and then passed into **[eval]** for evaluation.

The two Active Events **[for-each]** and **[set]** are declared in the p5.lambda project, and as you can
see in its code, they're actually quite simple Active Events, and nothing you cannot implement yourself,
to extend the programming language if you wish. The third Active Event called **[create-widget]** is
declared in the p5.web project, and simply creates an Ajax widget of some type for you.

What happens when you evaluate the above piece of Hyperlambda, is that **[eval]** is being invoked,
with the root node's children of your above Hyperlambda, as its _"instructions"_ to evaluate. The children
nodes of this root node again, are simply references to Active Events, having the same names as their node
names, and are being raised sequentially in order of appearance. To evaluate **[eval]** directly yourself,
is quite simple, and can be achieved with the following code

```hyperlambda
.x
  create-widget
    position:0
    element:h1
    innerValue:foo bar
eval:x:/@.x
```

The above code creates a node structure, or a _"data segment"_, containing lambda instructions, which are
evaluated as you invoke **[eval]** on the **[.x]** node.

Notice that **[eval]** will _not_ evaluate any nodes, having names, starting with an underscore (\_) or a
period (.). This allows you to create data-segments in your code, where these data-segments will not be
raised as Active Events. All nodes having a name, not starting with an underscore or period, will be raised
as Active Events by **[eval]**. This might produce weird results for you, if you do not start your
data-segments with an underscore, and there just so happens to exist an Active Event, with the same name
as the name of your data-segment. A good rule of thumb, is to always start your data-segments with an
underscore for the above reasons.

Notice though that if you explicitly invoke **[eval]** on a node, that starts with an underscore (\_)
or period (.), then **[eval]** will **evaluate that node's children nodes**. There also exists an
**[eval-mutable]** Active Event, which allows you access to the entire main root graph object from within
your lambda - But this Active Event is for the most parts for keyword implementors, and rarely something
you'd consume yourself from Hyperlambda.

**Notice**, **[eval]** creates a new root node hierarchy, not allowing the execution to gain access to
any nodes from the outside of itself, except nodes explicitly passed in as parameters. It also
**evaluates a shallow copy** of your lambda.

```hyperlambda
.x
  create-widget
    element:h1
    innerValue:x:/../*/argument?value
eval:x:/../*/.x
  argument:foo
eval:x:/../*/.x
  argument:bar
```

In the above code, logically the **[.x]** node is treated as an _"inline function"_, before it is invoked
twice, with different arguments. Notice that none of the arguments passed into **[eval]** are being raised
as Active Events, since the execution instruction pointer, starts out with an _"offset"_ being the number
of arguments you pass in. This makes it perfectly safe to pass in arguments to **[eval]**, without starting them
with an underscore or period. Notice also that **[eval]** doesn't actually execute the node(s) given itself,
but rather a shallow copy of these nodes. **[eval]** can also return values, such as below.

```hyperlambda
.x
  return:Hello World
set:x:/+/*/innerValue?value
  eval:x:/../*/.x
p5.web.widgets.create-literal
  position:0
  element:h1
  innerValue
```

To return more than one node, simply insert your return values into the root node, somehow, such as below
for instance.

```hyperlambda
.x
  return
    foo1:bar1
    foo2:bar2
eval:x:/../*/.x
```

Notice in the latter example, how there are two new children nodes of **[eval]** after it has evaluated.
**[eval]** is also extremely flexible, and allows for you to add, remove and change nodes, including at the
current instruction pointer, or just before it, or after it, if you wish. Consider the following code

```hyperlambda
insert-after:x:
  src
    create-widget
      element:h1
      innerValue:Foo bar
```

What happens in the above example, is that when **[eval]** comes to the **[insert-after]** Active Event invocation,
there exists no **[create-widget]** in the execution tree. The invocation to the **[insert-after]** Active Event
however, injects this node just after the **[insert-after]** node in the execution tree. When the instruction
pointer is done evaluating **[insert-after]**, it finds a new Active Event invocation, being our newly
added **[create-widget]** Active Event reference, which it in turn evaluates, resulting in creating our literal
widget.

If you change the above code to using **[insert-before]** instead, there will be no literal widget after
evaluation, but the node hierarchy is clearly changed, as you can see in the output.

```hyperlambda
insert-before:x:
  src
    create-widget
      element:h1
      innerValue:Foo bar
```

The instruction pointer for **[eval]**, managed to perfectly keep track, of where in the execution tree it
currently is, without messing up the order of your instructions. To prove it, run the following code, which
shows that only one widget is created.

```hyperlambda
insert-before:x:
  src
    create-widget
      element:h1
      innerValue:Foo bar
create-widget
  element:h1
  innerValue:Foo bar 2
```

For the record, the above `:x:` expression, is the _"identity expression"_ in the p5 expression engine,
and simply means the _"current node"_. This is where all expressions starts out.

### Evaluating multiple lambda objects in one go

**[eval]** can also execute multiple sources in one evaluation. Imagine you have a node hierarchy, where
you have several nodes you wish to evaluate. This can actually be done with one **[eval]** invocation, since
each result of your expression will be evaluated in order, according to how they were fetched by your expression.
Try out the following code.

```hyperlambda
.x
  create-widget
    element:h1
    innerValue:Widget no 1
.x
  create-widget
    element:h1
    innerValue:Widget no 2
eval:x:/../*/.x
```

The above expression will return the **[.x]** nodes in consecutive order, and **[eval]** will evaluate them
in the order returned by the expresssion. Hence, we end up with two new widgets on our page. Similar things
will happen if you have two results of an **[eval]** expression, returning some values back to caller, such
as the following.

```hyperlambda
.x
  return
    foo1:bar1
.x
  return
    foo2:bar2
eval:x:/../*/.x
```

Of course, if one of our **[eval]** invocations, returns a simple value, then only the latter's return value
will end up in our tree after evaluation. Imagine the following.

```hyperlambda
.x
  return:foo1
.x
  return:foo2
eval:x:/../*/.x
```

In the above scenarion, after evalution of our **[eval]**, only the _"foo2"_ string will be the value of our
eval invocation.

### Notice!

Nodes starting with either an underscore (\_) or a period (.), are _impossible_ to evaluate with **[eval]**.
Hence, Active Events starting with one of these characters, are considered _"protected events"_, and only
C# code can raise these. Realize though, you can evaluate an **[eval]** block, which has a name, starting
with an underscore or period. However, the **[eval]** block won't raise any nodes as Active Events that are
starting with either of these two characters itself. The convention is to use underscore (\_) for _"data segment"_,
and period (.) for _"lambda blocks"_, although you can choose this for yourself. There are no semantic
differences in whether or not you use an underscore or a period.

### Evaluating stuff that's not a node

If you try to evaluate something that is somehow not a node, then this object will be attempted converted into
a node, before it is evaluated. This allows you to evaluate strings, values, and even integer values for that
matter, as if they were a node structure, making **[eval]** perform an automatic conversion (attempt) on your
values before evaluating your object. Example below illustrates this.

```hyperlambda
.x:@"micro.windows.info:Hello World
return:Returned from string!"
eval:x:/../*/.x?value
```

You can still return values and nodes as usual. There is actually no difference in regards to logic of evaluating
a node or lambda object, or evaluating a string, except that unless the string you try to evaluate does not for
some reasons convert legally into a node, the Hyperlambda parser will choke, and throw an exception. In P5, you
could in theory, evaluate any block of lambda nodes you wish. Whether or not this makes sense though, depends
upon the nodes you try to evaluate.

### Evaluating a block of lambda instead of an expression

If you do not provide a value to **[eval]**, then it will instead of handling its children as arguments,
directly evaluate its children, as a lambda block. This is often useful in combination with for instance
the **[set]** and **[add]** Active Events, which we will later dive into, further down in the document here.
However, to illustrate its simplest version, imagine this code.

```hyperlambda
eval
  create-widget
    element:h3
    innerValue:Foo, bar!
```

In the above code, the p5.lambda execution engine will see that our **[eval]** invocation does not have any
expression, or anything in its value, that can be converted into a node in any ways. Therefor it will simply
evaluate its own children nodes, as a piece of lambda block, having still the root node of its evaluation
being the **[eval]** node itself. Later in our documentation we will dive into some use-cases where this is
extremely useful. But for now, just keep it at the back of your mind.

### [eval-whitelist] - Eval is not (necessarily) evil

There exists an overload of eval called **[eval-whitelist]**, which allows you to pass in a subset of legal
Active Events, as an **[events]** list. If either directly, or indirectly within this execution, any Active
Event not in this whitelisted **[events]** list is attempted to be raised - Then a security exception will be
thrown. This is highly useful for evaluating code you are uncertain of whether or not is safe, for some reasons.
Example is given below.

```hyperlambda
.x
  set:x:/..?value
    src:foo
  chokes
eval-whitelist:x:/@.x
  events
    set
    src
```

Notice in the above code, that regardless of that there actually doesn't exist a **[chokes]** Active Event
in your vocabulary, it will still try to raise this as an Active Event, and since it is not in the whitelist,
it will literally choke.

### "Keywords" in p5.lambda

p5.lambda contains several _"keywords"_, which aren't really keywords as previously explained, but in fact
Active Events, creating an extremely extendible environment for your programming needs. But the core
keywords, that should exist in a default installation of Phosphorus Five are as following.

### [eval-x], forward evaluating expressions

This Active Event allows you to forward evaluate expressions, and is useful when you want to use expressions,
in Active Event invocations, where the Active Event itself, does not in general accept expressions, or
you need to evaluate expressions for some other reasons, before the event is invoked. Consider the following
code for instance

```hyperlambda
_y:hello world
.x
  return:x:/../*/_y?value
eval:x:/../*/.x
```

At the point where **[.x]** is evaluated, it no longer has access to the **[\_y]** node, since this
node is outside the scope of our **[.x]** node. Now we could of course use **[eval-mutable]**, but
this would create potentially other problems. Among other things, having the evaluated code having
access to the entire execution tree. Instead we could choose to forward evaluate the expression inside
of our **[.x]** node, in our **[return]** invocation, such as the following code does.

```hyperlambda
_y:hello world
.x
  return:x:/../*/_y?value

// Forward evaluating the expression inside of [.x] first
eval-x:x:/../*/.x/*/return

// THEN we evaluate [.x]
eval:x:/../*/.x
```

When we then invoke our **[eval]** Active Event, the value of **[return]**, is no longer an expression,
but contains the constant value of that expression, as evaluated during our invocation of our **[eval-x]**
Active Event. Notice also that **[eval-x]** _always_ expects a Node result set in its expression,
and not a "?value" or "?name". This is because **[eval-x]** can only evaluate expressions, which can only
be found in values of nodes.

### [add], adding nodes to your trees

The **[add]** Active Event, allows you to dynamically add nodes into your lambda objects. This Active
Event must be given an expression as its destination, and can optionally take many different forms of
sources through its **[src]**. For instance, to add up a static source, into some node destination, you
could accomplish that doing the following.

```hyperlambda
.x
  foo1:bar1
add:x:/-
  src
    foo2:bar2
```

A static source, can be as complex as you wish, and contain any tree hierarchy you can possibly declare
in your lambda objects. As previously explained at the top of this document, modifying the instruction
pointer will work perfectly well, allowing you to create code that modifies itself, before it evaluates
the parts it notified/added to itself. Imagine the following code.

```hyperlambda
.x
  create-widget
    element:h1
    innerValue:Original, static widget
  add:x:/..
    src:x:/../*/argument/*
eval:x:/@.x
  argument
    create-widget
      element:h1
      innerValue:Dynamically injected into lambda object
```

What happens in the above lambda object, is that the **[eval]** invocation, passes in an **[argument]** argument,
which contains a **[create-widget]** invocation. After evaluating the first **[create-widget]** node, inside
of our **[.x]** node, the **[add]** invocation appends the content of our **[argument]** node, which was
passed into **[eval]** into the root node for our tree, which during the evaluation of **[.x]** is actually **[.x]**
itself. Then the instruction pointer moves on, realizing it has _another_ (newly dynamically added)
**[create-widget]** node, which is then evaluated finally, before the instruction pointer returns from **[.x]**.
This feature of course, is true for all Active Events that modifies the execution tree somehow. In the above code,
you also see how to create a dynamic source for your **[add]**, having an expression leading to the source,
and not a constant hierarchy.

For the record, the above features I think is something completely unique to P5, allowing injection of
additional code, into _"methods and functions"_ (lambda objects), through modifying the execution tree it
is currently executing directly, without any parsing of code occurring. This dynamic nature of lambda objects,
is as far as I know, completely unique to P5. Other languages can take anonymous delegates, and function lambda
objects - But modifying an existing function object, is completely unique to P5 as far as I know.

### [if], [else-if], [else] and [while]

These Active Events is what allows you to control the flow of your program, using what's traditionally referred
to as branching your code. Logically they work roughly the same as most other branching keywords you've seen
in other programming languages. With some subtle differences though. However, let's create the simplest if
lambda object we possibly can, to see them in action first.

```hyperlambda
if:bool:true
  create-widget
    element:h1
    innerValue:Yup, we branched!
```

The above **[if]** will yield true, since it is given a constant, having the value of boolean true. If we
exchanged the value to false, it would not create our widget. If you exchange the type declaration of
your object though, to string, which is the implicit type, with the following code, you would see something
different.

```hyperlambda
if:false
  create-widget
    element:h1
    innerValue:Yup, we branched!
```

This is because all string literals, implicitly converts to _true_ for your conditional Active Events. Think
of this as implicit conversion to boolean values, according to _"does there exist something"_. Meaning, not
equals null is the implicit logic of your conditional events. You could do this with any other types you wish,
for instance integer numbers too.

```hyperlambda
if:int:1
  create-widget
    element:h1
    innerValue:Yup, we branched!
```

The above code will even branch if you exchange the _"1"_ with a _"0"_, since there _"exists something"_,
which makes the condition yield true. Notice though, that if you remove the `:int:1` part above, and exchange
it with a _"null node value"_, the branch will still occur.

```hyperlambda
if
  create-widget
    element:h1
    innerValue:Yup, we branched!
```

The reason why the branch occurs, is because the conditional Active Events of P5, will if not given a condition
as their values, use the return value of the first child node, treated as an Active Event itself, to check if
the condition yields true. This allows you to embed event invocations, as the conditions to conditional events.
To understand how this works, try the folloing code.

```hyperlambda
if
  foo
  create-widget
    element:h1
    innerValue:Yup, we branched!
```

In the above code, the **[foo]** node, will be evaluated as an Active Event, and if it returns true (or some sort
of _'existence object'_) - Then the condition as a whole will yield true. If you created an Active Event
named **[foo]**, and you made sure it returned for instance _"true"_, then the branch would occur, and we
would have our widget created. Since **[foo]** above, is not an existing Active Event, it will not do anything,
making its value still being _"null"_ after evaluation of itself, meaning the condition for **[if]** yields false,
and no widget is created, since the branch does not occur. To use constants as your conditions for your
conditional events, often makes little or no sense. A more useful approach, would be to use an expression,
yielding the results of some node-set, such as the following.

```hyperlambda
_x:bool:true
if:x:/../*/_x?value
  create-widget
    element:h1
    innerValue:Yup, we branched!
```

If you change the value of the above **[\_x]** node to false, or _"null"_, the branch will not occur. If you
change it to for instance the integer 5, the branch will occur though, since the implicit conversion to boolean
will yield true. A more useful approach, would be to compare it against, either some sort of constant value,
or the results of another expression. Imagine the following code.

```hyperlambda
_x:int:5
if:x:/../*/_x?value
  =:int:5
  p5.web.widgets.create-literal
    element:h1
    innerValue:Yup, we branched!
```

In the above code, the return value of the expression in our if, will be compared against the constant of the
integer valur of 5, and the result will yield a match. Hence, the branch occurs. Notice, that branching
conditions in p5.lambda are type sensitive, which means if you run the following code, where you compare an
integer to a string, it will _not_ yield true, since their types are different.

```hyperlambda
_x:int:5
if:x:/../*/_x?value
  =:5
  create-widget
    element:h1
    innerValue:This won't show!
```

You could however type cast the results of your expression above, to make them become equal, like the
following code illustrates.

```hyperlambda
_x:int:5
if:x:/../*/_x?value.string
  =:5
  create-widget
    element:h1
    innerValue:Yup, we branched!
```

The _".string"_ parts at the end of our expression above, will convert the results of our expression,
into a string, which of course will create a match for our conditional statement, since this value of
our expression, will equal the constant string value of _"5"_ in our '=' condition. There exists 8
comparison _"operators"_ in p5.lambda. "=", "!=", ">", "<", ">=", "<=", in addition to two like comparison
operators "\~" and "!\~". The first 6 of these _"operators"_, does what you'd expect them to do,
and works the same way in most other programming languages - While the two latter _"like comparison"_
operators, works the same way as the tilde (\~) works in expressions, and are logically the equivalent
of asking _"does this string exist in the condition"_. Imagine the following code as an illustration.

```hyperlambda
_x:thomas hansen
if:x:/../*/_x?value
  ~:hansen
  create-widget
    element:h1
    innerValue:Yup, we branched!
```

The above condition will yield true, since the string of _"hansen"_ do exist in the return value of
the initial expression. Notice, that this is a case sensitive string comparison, and only works for
string comparisons. Or to be more precise, the values of both the constant, and the expression, will
be automatically converted to strings before they're compared for a match. For the record, all
comparison operators are actually simply Active Events, and you can easily create your own extension
comparison operators, creating your own Active Events. If you wish to do this, then make sure you return
true from your Active Event if it is supposed to evaluate to true.

### More conditions, compound comparisons

In addition to these comparison operators, there is also all four boolean algebraic operations. Allowing
you to do things logically equivalent to "if x equals some-value _AND_ y not-equals some-other-value"
for instance. Imagine the following code.

```hyperlambda
_foo1:bar1
_foo2:bar2
if:x:/../*/_foo1?value
  =:bar1
  and:x:/../*/_foo2?value
    =:bar2
  create-widget
    element:h1
    innerValue:Yup, we branched!
```

In the above code, both the conditions for the **[if]** and the **[and]** comparisons, must yield true,
for the condition as a whole to yield true. Meaning, if you change for instance the **[\_foo2]**'s value
to _"bar3"_, the branch will not occur. This allows you to create compound conditions, where the entire
condition as a whole must yield true, for the branch to occur.

The 3 basic algebraic operations are as following

* __[and]__
* __[or]__
* __[not]__

Each boolean operator, must be a direct child of the comparison it is supposed to algebraicly fuse with.
This becomes the equivalent of paratheses in traditional programming languages, allowing you to scope
together conditions, the same way you'd use parantheses in other languages. Imagine the following code
for instance.

```hyperlambda
_name:thomas
_surname:hansen
if:x:/../*/_name?value
  =:thomas
  and:x:/../*/_surname?value
    =:hansen
  or:x:/../*/_name?value
    =:john
    and:x:/../*/_surname?value
      =:doe
  create-widget
    element:h1
    innerValue:Yup, we branched!
```

In the above example, the branch will only occur if either the **[\_name]** is _"thomas"_ **and** the
**[\_surname]** is _"hansen"_ - _OR_ the **[\_name]** is _"john"_ and the **[\_surname]** is _"doe"_.
If the values are for instance _"thomas"_ and _"doe"_, then the branch will not occur. This is because
the last **[and]** operator, is a child of the **[or]**, and hence these are scoped together, the same
way they'd normally be using parantheses in other programming languages. The above is the equivalent
of `if ((_name == "thomas" && _surname == "hansen") || (_name == "john" && _surname == "doe"))` in for
instance C#.

If you have both **[and]** and **[or]** operators, in the same scope, then **[and]** will get presedence,
as in other programming languages.

Notice though that the syntax for branching in P5, creates verbose code for complex conditions. This
is due to that Hyperlambda is a simple key/value/children file format, and in fact not a programming
language per se. This means that each node can only have one name, and one value. In addition to a
list of children. This has the benefit though of being able to logically traverse your branching code,
in code, understanding its conditions, without having to do any parsing of conditional expressions.

This is intentionally created this way, to allow for intelligently and semantically make it possible
for the machine itself, to understand the code from a semantic point of view, before it evaluates
the lambda objects.

So even though this sometimes creates relatively verbose code, compared to other less verbose
programming languages, it also has some pretty nice side-effects and features - Such as making it very
easy for your machine to understand what it is evaluating, by intelligently traversing the execution
tree, before it evaluates your code. Allowing for it to modify the execution tree, directly itself,
before it evaluates your lambda object etc.

However, the negative side-effect, is that sometimes the code becomes more verbose and longer,
for tasks that in other programming languages would require far less code. On average though,
the amount of code necessary to create some piece of logic, would often be far less in P5, than
what it would be in other programming languages. The above code example though, is the trade-off
price you pay for this feature.

To understand why, realize that there literally is no programming language, only the execution tree -
So what you see in P5, is what a compiler, or an interpreter would produce, after having parsed your
code. In P5, _"code"_ does not pre se exist, and you are modifying the _execution tree_ directly,
without any steps inbetween you and your machine, as would be the case in most other programming languages.

Some of the consequences of this, is that it to a much larger extent, allows the machine to produce
its own code. In the long run, this facilitates for off-loading the burdon from the programmer,
creating a much more _"intention driven"_ environment for the programmer, where the programmer can
simply state his intentions, and the machine creates the code that implements these intentions.

If the above paragraphs sounds like lots of mumbo jumbo to you, simply ignore it, run with P5,
and watch your future change to the better - Making you more productive, as you start realizing
how you can reuse code, to an extent difficult to imagine before you've seen it for yourself.

### [not]

The **[not]** operator, due to the above reasoning, is slightly special in P5. This is because
it needs to be the name of a node. Hence, it will as a result always have to _follow_ the condition
it is supposed to negate. Let us illustrate with an example.

```hyperlambda
_x:bool:false
if:x:/../*/_x?value
  not
  create-widget
    element:h1
    innerValue:Yup, we branched!
```

In the above example, the return value of the expression in the **[if]** invocation, would normally
return _"false"_. However, this value is negated due to the **[not]** boolean we're adding up on the
line afterwards. Hence, the condition as a whole, will in fact yield _"true"_, and the branching
occurs, even though the result of the expression in our if actually is true. **[not]** will hence
negate the results of its previous condition.

You can also **[not]** more complex conditions, such as the following is an example of.

```hyperlambda
_x:thomas
if:x:/../*/_x?value
  =:thomas
  not
  create-widget
    element:h1
    innerValue:Yup, we branched!
```

The above branch would normally occur, since the constant of _"thomas"_ in our **[\_x]** node's value,
and the constant of our **[=]** operator would yield true. However, since the entire comparison is
negated due to our **[not]** operator at the end of our comparison, the branch does not occur,
since the _"true"_ value of our comparison, becomes negated to _"false"_, yielding false as the product
of our condition as a whole.

The **[not]** operator, should always be _the last operator_ in the scope where it appears. It cannot
take any arguments - Neither constants, nor expressions.

### [else-if] and [else]

The **[else-if]** is similar to the **[if]** Active Event, except it will only evaluate its condition(s)
if the matching **[if]** yields false. **[else]** will evaluate to true, if all of its matching and
previously declared **[if]** and **[else-if]** before it evaluated to false.

### [while]

The **[while]** Active Event, is similar to the **[if]** and **[else-if]**, except it will check its
conditions upon every iteration. And only as the condition somehow yields false, it will break out
of its scope, stopping the loop, and evaluate the next node in its scope. A **[while]** loop, can
be explicitly stopped, using the **[break]** Active Event. Below is an example.

```hyperlambda
_data
  foo1
  foo2
  foo3
while:x:/@_data/0
  create-widget
    innerValue:x:/@_data/0?name
  set:x:/@_data/0
```

Notice that **[while]** will not iterate more than 5,000 iterations by default. If you remove the
last **[set]** invocation above for instance, you will have an exception after 5,000 iterations.
Try out the following code.

```hyperlambda
_data
  foo1
  foo2
  foo3
while:x:/@_data/0
  create-widget
    innerValue:x:/@_data/0?name
```

The above code will throw an exception. This is to safeguard against infinite loops, which would
make your CPU end up stalling, if a logical bug existed in your code. You can override this behavior
though, by adding up **[\_unchecked]** as an argument to your **[while]**. If you did, you could
iterate for as long as you wish, but this would also possibly create infinite loops for you,
unless you're careful with your code.

### [for-each], iterating over a resulting node-set

The **[for-each]** Active Event, will iterate once for each resulting node/value/name in its expression.
For each iteration will inject a **[\_dp]** node as its child, having the value of whatever is iterated
over. For instance, consider the following code.

```hyperlambda
_data
  foo1:thomas
  foo2:hansen
for-each:x:/@_data/*?value
  eval-x:x:/+/*
  create-widget
    element:h1
    innerValue:x:/@_dp?value
```

The above code, will iterate the values of each children node of the **[\_data]** node, creating one
literal widget for each iteration. If you add more nodes beneath the **[\_data]** segment, you will
have more iterations, and hence more widgets in your page. Notice that the **[\_dp]** is special,
according to what type of result-set you're iterating over. If you iterate over a node, instead of a
value and/or a name as we do above - Then the **[\_dp]** node will store the currently iterated node
by reference, in the value of itself. To illustrate with some code.

```hyperlambda
_data
  foo1:thomas
  foo2:hansen
for-each:x:/@_data/*
  set:x:/@_dp/#?value
    src:COOL GUY!!
```

If you watch the output of the above code in for instance Hypereval, you will see that after evaluation,
all values of the children nodes of **[\_data]** will become _"COOL GUY!!"_ after evaluation. This is
because we're iterating these nodes _"by reference"_. The "#" (hash character), is a special iterator
in your expressions, yielding the value of the currently iterated node, as a node by itself. Since this
node is passed into our **[for-each]** by reference, we can change this node in our **[set]** invocation.

To summarize; If you're iterating a value or name, the iterated value will be in the value of the **[\_dp]**
node. If you're iterating a node-set in your expressions instead, then the node will be passed in by
reference to your **[for-each]**, through its **[\_dp]** node. The **[\_dp]** node, is dynamically injected
into your lambda object for each iteration of your result set. Notice that the results of your expression,
is evaluated as a copy of the resulting nodes, allowing you to inject nodes into your result set, without
entering an inifinite loop. The following illustrates that fact.

```hyperlambda
_data
  foo1:thomas
  foo2:hansen
for-each:x:/@_data/*
  insert-after:x:/@_dp/#
    src
      is:cool
```

Unless we had evaluated a copy of the results of our above expression above, we would have entered an
infinite loop. **[for-each]** will never enter an infinite loop.

### [insert-before] and [insert-after]

These two Active Events works similar to the **[add]** Active Event, except instead of adding to a
node's children collection, they will either insert before or after whatever node(s) you're pointing
to in their destination expressions. Imagine the following code.

```hyperlambda
_data
  foo1:thomas
  foo2:hansen
add:x:/@_data/*/foo1
  src
    this-was-add
insert-before:x:/@_data/*/foo1
  src
    this-was-insert-before
insert-after:x:/@_data/*/foo1
  src
    this-was-insert-after
```

The above will result in the following.

```hyperlambda
_data
  this-was-insert-before
  foo1:thomas
    this-was-add
  this-was-insert-after
  foo2:hansen
/* ... rest of your code ... */
```

Besides from this, **[insert-before]** and **[insert-after]** works identical to **[add]** in regards to
parameters they can take, and how they treat these parameters.

### [set]

The **[set]** Active Event requires an existing node, which it will modify somehow. You can set the name,
value or the node itself as you please. The **[set]** Active Event, is in these regards probably the most
basic and fundamental Active Event in P5. It becomes the equivalent of assignment in traditional programming
languages. Imagine the following code.

```hyperlambda
_data:foo
set:x:/-?value
  src:bar
```

As previously said, **[set]** can also change names of nodes, and it can also change the nodes themselves,
such as this example illustrates.

```hyperlambda
_data:foo
set:x:/-
  src:node:"howdy:world"
```

After evaluating the above lambda, the results will resemble the following.

```hyperlambda
howdy:world
set:x:/-
  src:node:"howdy:world"
```

Notice that the **[\_data]** node is now completely gone, and replaced with a **[howdy]** node,
having the value of _"world"_. This also works for entire node hierarchies, and regardless of
how many children the original **[\_data]** node had, the entire node, including its children,
would be replaced by the above **[src]** object.

Notice in the above code, that the static **[src]** declaration of **[set]**, is actually type
declared as a `node`. Meaning, the value of **[src]**, is itself a node, with the name of _"howdy"_
and the value of _"world"_. If it wasn't a node, **[set]** would try to convert it to a node, since
the target destination is a node itself.

**[set]** can also change the names of nodes, if the destination expression is type declared as `?name`.
If you want to remove a node, or remove a value for that matter, then you can use **[set]** without a
source at all. This will remove the destination, instead of changing it. For instance to remove a node
you could use something like the code below.

```hyperlambda
_data:foo
set:x:/-
```

Notice that the lambda execution engine perfectly tolerates the above code, without cluttering
the execution instruction pointer, or messing up the order of the nodes that are supposed to be
evaluated. The same way it'll do if you add or change nodes just below its current instruction pointer.

The lambda execution engine is extremely tolerant in regards to changes in the tree. You can even
remove the currently iterated pointer, and still have the engine perfectly continue its normal execution
flow. Consider the following.

```hyperlambda
_data
set:x:
set:x:/@_data?value
  src:foo
```

If you run the above code, it'll perfectly evaluate, and change the **[\_data]** node's value to _"foo"_.
Realize though, that since the tree is changed after our first **[set]** invocation, expressions might
change as a result. Consider this code for instance.

```hyperlambda
_data
set:x:
set:x:/-2?value
  src:foo
```

The above code have some _"weird consequences"_. To understand why, realize that as the execution
engine reaches your second **[set]** invocation, the first **[set]** is gone. Hence, the
"2nd younger sibling iterator" (`/-2`) will not return the **[\_data]** node, but instead traverse past the
beginning of the tree. Realizing it's done a round-trip, wrapping around to the eldest node in the tree,
ending up with having the value of the second **[set]** node, changing its value, and not the **[\_data]** node
as might be assumed at first glance. This occurs since expressions are not evaluated before the instruction
pointer reaches them. Which allows you to reference nodes in your code, which are later created, as a
consequence of the code being evaluated. Logically we say that _"expressions are lazily evaluated"_.

### Using an Active Event source for your [set], [add] or [insert-x] invocations

Both the **[set]**, **[add]** and **[insert-xxx]** Active Events can instead of taking a static source,
use an Active Event as their source. Imagine the following.

```hyperlambda
_data
  person
    first:Thomas
    last:Hansen
  person
    first:John
    last:Doe
.exe
  return
    full-name:foo
add:x:/@_data/*
  eval:x:/@.exe
```

In the above code, the **[.exe]** lambda object will be invoked once for each destination of our **[add]**.
However, our **[eval]** will be invoked, before the **[add]** tries to determine what its source actually is.
You can use any Active Event you wish instead of a **[src]** node, including your own custom Active Events.

### [switch], a shorthand for multiple if invocations

If you have a bunch of values, which you want to compare some value towards, then you can use a **[switch]**
statement, such as the below example illustrates.

```hyperlambda
_foo:thomas
switch:x:/-?value
  case:john
    micro.windows.info:Hi John
  case:jane
    micro.windows.info:Hi Jane
  case:thomas
    micro.windows.info:Hi Thomas
  default
    micro.windows.info:Hi stranger
```

In the above example, unless the name is either _"thomas"_, _"jane"_ or _"john"_ in the value of the **[\_foo]**
node - Then our **[default]** lambda block will be evaluated. If the name is _"thomas"_, then the first **[case]**
block will be evaluated, etc. This allows you to more easily create complex branching lambda objects, where you
are checking the same expression, for multiple results. The **[default]** block is what is evaluated, if none
of the other alternatives was a match.

In a **[switch]** Active Event invocation, you can also create what's commonly referred to as _"fall through"_.
By having no block in one or more of your **[case]** nodes, the lambda block of the next **[case]**,
that has a lambda block, will be evaluated instead, if the case value was a match. Consider the following.

```hyperlambda
_foo:john
switch:x:/-?value
  case:john
  case:jane
    micro.windows.info:Hi John/Jane
  case:thomas
    micro.windows.info:Hi Thomas
```

In the above example, the lambda block of _"jane"_ will evaluate, for both the value of _"jane"_, and the
value of _"john"_. As you can see in the above example, the **[default]** block is optional, and excluded
in the above example. If you provide a **[default]** block, you should put it as the last block in your
**[switch]** lambda.

### [try], [catch], [finally] and [throw]

No programming language is complete, without at least some sort of basic exception handling. P5 support
exceptions through these Active Events.

* __[try]__
* __[catch]__
* __[finally]__
* __[throw]__

They do what you'd expect them to do from other programming languages. The **[try]** Active Event creates
a safe block, where no exceptions will penetrate through, without invoking any coupled **[catch]**, and/or
its **[finally]** block. The **[throw]** Active Event, raises an exception, unwinds the stack, all the way
to the next **[catch]** block, evaluating the lambda code inside of that catch if existing. The **[finally]**
block evaluates, regardless of whether or not an exception occurs. If an exception occurs, the exception
is re-thrown after evaluation of the **[finally]** block though, unless a **[catch]** block also exists.
Consider the following.

```hyperlambda
try
  throw:Darn it, my head hurts!!
catch
  micro.windows.info:Oops ...!!
```

In the above example, we explicitly throw an exception, with the message of _"Darn it, my head hurts!!"_.
If you wish, you can retrieve this error message from inside of your **[catch]** block, through a **[message]**
argument passed into your **[catch]**. To illustrate this with some code.

```hyperlambda
try
  throw:Darn it, my head hurts!!
catch
  micro.windows.info:x:/@message?value
```

In addition to the **[message]** argument passed into your **[catch]** block, you also get the **[type]**
of exception thrown, which is the fully qualified class name of the class from .Net/Mono, and the
entire **[stack-trace]**. The type of an exception thrown from Hyperlambda using **[throw]**, will always
be `p5.exp.exceptions.LambdaException`. When you **[throw]** an exception from Hyperlambda, you can also
pass in an expression as its message.

### [return]

Functions or methods does not really exist in P5. However, you can treat any lambda object, or node structure,
as if it was a _"function"_, and invoke it using for instance **[eval]**. Sometimes when you do, you might
want to return early from the evaluation of your lambda objects. This can be accomplished with the **[return]**
Active Event. This Active Event will return whatever you choose to return to the caller of the lambda evaluation,
if any, and end the execution of the outer most lambda block. To illustrate with some code.

```hyperlambda
.exe
  return:foo
  micro.windows.info:Never evaluated
eval:x:/-
```

After evaluating the above code, the value of the **[eval]** node will contain the string of _"foo"_. In addition,
the **[micro.windows.info]** invocation will never be invoked, since when the evaluation of **[return]** is done,
no further evaluation inside of the inner most evaluated scope will be done. Since the outer most evaluated scope
in the above example, is the evaluation of the **[.exe]** node, this means that the control will be returned
back to the node after our **[eval]**, invoking the **[.exe]** node. Since there is no more nodes though,
execution of our lambda as a whole exits. You can also return multiple nodes back to caller, and complex
tree structures, using something like this for instance.

```hyperlambda
.exe
  return
    foo1:bar1
      foo2:bar2
    foo3:bar3
eval:x:/-
```

After evaluation of the above code, every child node of the above **[return]** invocation, will be appended
into the **[eval]** node that invoked our **[.exe]** block. Notice, you don't need to return anything.
You can of course also use a simple empty **[return]**, to avoid further execution of your lambda block.

### [break]

**[break]** works similar to the **[return]** Active Event, except it only carries meaning inside of either
a **[while]** or a **[for-each]**. Instead of returning though, it stops all further execution of all code
within the first ancestor **[for-each]** or **[while]** block, and returns control back to the next sibling
node, after the loop node currently being breaked. Imagine the following code.

```hyperlambda
_data
  foo1:bar1
  foo2:bar2
  foo3:STOP
  foo4:bar4
_dest
for-each:x:/@_data/*
  if:x:/@_dp/#?value
    =:STOP
    break
  add:x:/@_dest
    src:x:/@_dp/#
```

The above code iterates the **[\_data]** segment's children, appending each child node into **[\_dest]**,
stopping further iteration if it encounters a value of _"STOP"_. After evaluation of the above lambda,
the **[\_dest]** node will contain the _"bar1"_ node, and the _"bar2"_ node, but none of the other two
nodes from the **[\_data]** segment. **[break]** works similarly for **[while]** loops. Below is an example.

```hyperlambda
_data
  foo1:bar1
  foo2:bar2
  foo3:STOP
  foo4:bar4
_dest
while:x:/@_data/0
  if:x:/@_data/0?value
    =:STOP
    break
  add:x:/@_dest
    src:x:/@_data/0
  set:x:/@_data/0
```

### [continue]

The **[continue]** Active Event will instead of breaking the entire loop, stop further execution of its
current iteration, and continue on the _next_ iteration for the loop. Imagine the exact same code as from
our **[break]** example, except we use **[continue]** instead.

```hyperlambda
_data
  foo1:bar1
  foo2:bar2
  foo3:STOP
  foo4:bar4
_dest
for-each:x:/@_data/*
  if:x:/@_dp/#?value
    =:STOP
    continue
  add:x:/@_dest
    src:x:/@_dp/#
```

If you execute the above code, you will have the **[foo4]** node appended into your **[\_dest]** node,
because this time, instead of stopping all further execution of your loop, the **[continue]** invocation
instead simply stops the _currently iterated execution_, and _continues_ with the next iteration.

### [apply]

This is a fairly advanced Active Event, and not necessary to understand Phosphorus Five. If it's hard to
understand, feel free to simply skip it. It is rarely if ever used in any of the core projects of P5.

With the **[apply]** Active Event, you are able to apply one **[src]** node, expression or constant, to your
destination, acccording to a **[template]**, referencing nodes and values from your **[src]**. It is kind
of the superman version of add. Let's look at an example.

```hyperlambda
_people
  thomas
  john
  jane
apply:x:/../*/create-widget/*/widgets
  src:x:/../*/_people/*
  template
    literal
      {innerValue}:x:?name
create-widget
  widgets
```

To understand the above code, realize that the **[apply]** Active Event will iterate over its **[src]**
expresssion node-set, and apply the **[template]** for each iteration into its destination expression,
which is the value of the **[apply]** node itself. While each node, having a name surrounded with braces
"{}" inside of your **[template]**, will be expected to be dynamically fetched from the **[src]**, which
must be a relative expression, relative to the currently iterated **[src]** node-set.

Since the **[src]** in the above example, is iterating each children node beneath the **[\_people]** node,
this means that the `:x:?name` expression, for the above **[{innerValue}]** node, will fetch the names of
these nodes (_"thomas"_, _"john"_ and _"jane"_) as the loop iterates.

The **[apply]** event is therefor quite useful for iterating nodes, where you wish to apply some parts
of the results from your iteration, into some destination. In fact, that last sentence largely defines
its purpose. Whenever you want to loop and add, you could probably use **[apply]** instead. It sometimes
helps to unroll one iteration in your mind, to understand the expressions inside of a **[template]** argument.
The apply Active Event is arguably the _"Superman version"_ of **[add]**.

In the above example, our **[src]** expression yields 3 nodes as its result. Hence, the **[template]**
is applied 3 times to the destination, which is the value expression of our **[apply]** node. Every time
it applies the template, it will unroll each databound expression, such as the one in **[{innerValue}]**
above, with the expression being evaluated relatively to the currently iterated **[src]**.

Only nodes that have names surrounded by braces `{}` will have their values dynamically changed, according
to the **[src]** node being iterated. All other nodes and expressions, will be left unchanged after evaluation.
However, the braces `{}`, will be removed after **[apply]** has been evaluated. This means, that with the
above syntax, you can only change _values_ of nodes. However, to change and/or create new nodes using
**[apply]**, is achieved using another feature of **[apply]**. Before we look at that, let us first create
a more complex apply evaluation to further visualize the iteration of an **[apply]** invocation.

```hyperlambda
_people
  person1
    name:Thomas Hansen
  person2
    name:John Doe
  person3
    name:Jane Doe
apply:x:/../*/create-widget/*/widgets
  src:x:/@_people/*
  template
    literal
      {innerValue}:x:/*/name?value
create-widget
  widgets
```

The above p5.lambda, will produce roughly the same output. However, now you can more easily see how the
expressions inside of the databound nodes (nodes having names surrounded by braces ({})), are dynamically
fetching their values, having expressions being relative to the currently iterated node, since you see
the expression in the above **[{innerValue}]** node, is referencing a child **[name]** node.

Normally an identity expression will point to the node where the expression is declared. In **[apply]**,
the identity expression will instead lead to the currently iterated node, from its currently iterated **[src]**
node-set value.

If this was the complete feature list of **[apply]**, it wouldn't be very powerful though. First of all,
we cannot decide what names nodes are created dynamically. The **[template]** seems to be static for all
items. Secondly, we cannot return different nodes, according to the data result-set currently being iterated.
This too though is easily accomplished using **[apply]**. To understand how, realize that you can
invoke any Active Event you wish during an apply iteration, including also for instance **[eval]**.
This allows you to evaluate some lambda object, for each iteration of an apply iteration, passing in
the currently iterated source node. This is being done by prepending an alpha character in front of the
name of your databound node, at which point your entire databound node will be discarded, and replaced
by whatever (if anything), returned from the evaluation of your lambda object. Let us illustrate with some code.

```hyperlambda
_people
  person1
    name:Thomas
    surname:Hansen
  person2
    name:John
    surname:Doe
  person3
    name:Jane
    surname:Doe
.exe
  eval-x:x:/+/*/*/innerValue
  return
    literal
      innerValue:{0}, {1}
        :x:/@_dn/#/*/surname?value
        :x:/@_dn/#/*/name?value
apply:x:/../*/create-widget/*/widgets
  src:x:/@_people/*
  template
    {@eval}:x:/@.exe
    hr
create-widget
  widgets
```

Basically, to invoke an Active Event for a databound node, use syntax like this; `{@some-active-event}`.
The Active Event you invoke, can be any Active Event you wish. And whatever your Active Event returns as
nodes, will replace the databound node entirely after evaluation of your Active Event. If you watch the
output of the above code, you will see that there are no traces of our original **[{@eval}]** node after
evaluation. You can combine Active Event invocation sources, with plain databound sources, as we started
out with, and/or static nodes, as you see with the **[br]** node in the above example.

Notice that although we are creating a widget hierarchy above, you can create any node structure you wish
with **[apply]** - Including insertions into your database, etc. Any node structure you want to create,
is easily accomplished using **[apply]**. Databinding using **[apply]** in P5, is not in any ways
restricted to graphical objects, such as you're probably used to with databinding from other programming
languages.

#### Escaping databound template value

Sometimes, you have a node, which for some reasons, should be named **[{xxx-something}]**. This creates
a problem, since it'll be assumed to be a databound node. Use cases for such scenarios, might be for
instance nested **[apply]** invocations. For such cases, all you need to do, is to for instance prepend
your node's name with a back slash (\). Imagine the following code, which contains nested apply lambda
blocks.

```hyperlambda
_data
  person:1
    first:Thomas
    last:Hansen
  person:2
    first:John
    last:Doe
  person:3
    first:Jane
    last:Doe
_output1
_output1-exe
  eval-x:x:/+/*
  return
    innerValue:{0}, {1}
      :x:/@_dn/#/*/last?value
      :x:/@_dn/#/*/first?value
apply:x:/@_output1
  src:x:/@_data/*
  template
    create-widget
      {@eval}:x:/@_output1-exe
      onclick
        {_foo}:x:/*/first?value
        apply:x:/../*/_exe
          src:x:/../*/_foo
          template
            \{micro.windows.info}:Hello {0}
              :x:?value
        _exe
        eval:x:/-
eval:x:/../*/_output1
```

The above code might seem difficult to understand when you first look at it. The important point though,
is that we have two nested **[apply]** invocations above. Unless we had added the back-slash at line 29,
then this databound node would have been databound in our first apply invocation. Which obviously was
not our intention. Hence, we add a back-slash in front of it, which will be removed during the first
databound, and hence make sure our node is not databound during the first apply invocation.

The code creates one **[create-widget]** invocation, for each child node of our **[\_data]** segment,
having the **[innerValue]** created as a combination of the **[last]** and **[first]** name of our persons.
Then we handle the **[onclick]** Ajax event for our widgets, from where we invoke **[apply]** towards
an invocation of **[micro.windows.info]**. Our info window is then databound towards the **[\_foo]**'s value,
which was previously databound in our first **[apply]** invocation towards only the **[first]** name -
Resulting in that when our widgets are clicked, they will speak _"Hello 'first-name'"_.

The above is probably not a relevant example, and the same result could have been easily achieved with
other means. However, it is an example of how you must escape your inner **[apply]** invocations databound
nodes, if you embed them inside of outer **[apply]** invocations.

If you remove the back-slash at line 29 in the above code, instead of showing the first name in an info window,
it will show the ID of your person (1, 2 or 3) when you click the widget. This is because the
**[micro.windows.info]**, will be databound byt the first invocation to our **[apply]** Active Event,
which obviously was not our intention.

**Notice** - Sometimes you need to have the results of a **[apply]** invocations create a node who's
name starts with a alpha character (@). If the value of this node is databound itself though, this
would instead of creating a node starting with @ in its name, expect an Active Event who's name was
that of your node. You can accomplish this though creating a lambda object, returning a node, who's
name starts with an "@" though - And then use **[eval]** as an Active Event invocation for your
databound node. Imagine the following code.

```hyperlambda
_people
  person1
    name:Thomas Hansen
  person2
    name:John Doe
  person3
    name:Jane Doe
_exe
  eval-x:x:/+/*
  return
    @name:x:/@_dn/#/*/name?value
_out
apply:x:/@_out
  src:x:/@_people/*
  template
    {@eval}:x:/@_exe
```

#### Warning!

The **[apply]** Active Event is extremely powerful. But this power, comes with a cost, which is that it
is very easy to create extremely difficult to understand and obfuscated code - Unless you are careful.
Understanding how **[apply]** works, also requires some highly developed visualization skills.

### [fetch]

The **[fetch]** Active Event, allows you to declare a piece of lambda, have it evaluated, and _"fetch"_
some parts of its result into the value of your **[fetch]** node. This is highly useful when you for
instance have a **[set]** of **[if]** invocation, where you have a piece of code, that is to be evaluated,
for then to use one or more nodes deep inside of the result of your lambda evaluation, being used as the 
source for your **[if]** or **[set]** invocation. Consider the following code for instance.

```hyperlambda
.exe
  return
    person
      name:Thomas
      surname:Hansen
fetch:x:/0/0/*/surname?value
  eval:x:/@.exe
```

After evaluation of the above code, the value of **[fetch]** will be _"Hansen"_. This allows you to
pre-fetch one or more node's values, which again plugs perfectly into concepts such as **[if]** and
similar Active Events, expecting a node's value, and not a complete node hierarchy. **[fetch]**
is kind of like **[eval]** combined with a fetch operation, fetching another node's value, after
having evaluated its lambda. Consider this for instance, where we have injected an if inbetween
our root node and our **[fetch]** node.

```hyperlambda
.exe
  return
    person
      name:Thomas
      surname:Hansen
if
  fetch:x:/0/0/*/surname?value
    eval:x:/@.exe
  =:Hansen
  micro.windows.info:Hello family member
```

What happens in the above lambda, is that first the **[if]** Active Event is invoked. Since our if
node does not have a value, its first child is considered to be _"what is compared on our left hand side"_
parts. The logic of **[if]** is that if there's an Active Event invocation used to retrieve the left-hand-side,
then this Active Event is invoked before the two sides are compared for equality, as we're doing in the
above example. This raises our **[fetch]** Active Event.

The **[fetch]** Active Event again, will evaluate its children, almost like an **[eval]** invocation would -
With one crucial difference, which is that after evaluation of its block, the expression in the value
of **[fetch]** will be evaluated, and the results of that expression, will become the value of our **[fetch]**.

After the fetch has been evaluated, the comparison in the **[if]** occurs. Since the value of fetch now
is _"Hansen"_, and this value will be compared against the constant of _"Hansen"_, our evaluation yields true,
and **[if]** allows branching of evaluation to enter into its lambda block, evaluating our **[micro.windows.info]**
Active Event.

Intelligently using **[fetch]**, allows you to nest logical pieces of code, in a more logical way, the same
way you would using functions and methods in traditional programming languages.

Since **[fetch]** will put the results of its expression into its own value after evaluation, you can also
use it in combination with for instance **[add]** and the **[insert-xxx]** Active Events. The following
code illustrates this.

```hyperlambda
_exe
  return
    person
      name:Thomas
      surname:Hansen
      address:Foo str. 24
_result
add:x:/@_result
  fetch:x:/0/0/*/name
    eval:x:/@_exe
```

The above code of course will **[fetch]** a node, which will become the source for our **[add]**.

### [sort]

In P5 there is an Active Event which allows you to sort your nodes, and supply your own sort callback.
Your callback will be invoked with an **[\_lhs]** and an **[\_rhs]** node, asking you to determine which
node comes before the other of these two given arguments. Both the **[\_lhs]** and the **[\_rhs]** node
passed into your lambda block, will be passed in by reference, allowing you to traverse the tree, both
up and down from the nodes you wish to sort on. Imagine the following code, and let us sort them according
to their names.

```hyperlambda
_data
  person:1
    name:Thomas Hansen
  person:2
    name:John Doe
  person:3
    name:Abraham Lincoln
sort:x:/@_data/*
  if:x:/@_lhs/#/*/name?value
    <:x:/@_rhs/#/*/name?value
    return:int:-1
  else-if:x:/@_lhs/#/*/name?value
    >:x:/@_rhs/#/*/name?value
    return:int:1
  else
    return:int:0
```

After evaluating the above lambda, _"Abraham Lincoln"_ will be the first child beneath **[sort]**. Then you
will find _"John Doe"_, before finally _"Thomas Hansen"_. This is the alphabetical sort order of them,
according to their names.

Notice that your **[sort]** callback expects you to return either -1, 1 or 0. -1 for those cases when **[\_lhs]**
should come before **[\_rhs]**, 1 the opposite, and 0 for when the nodes are supposed to be handled as equals.
Also notice that **[sort]** does not change the original node-set, but handles it immutable, and returns
the sorted nodes as children of sort itself after evaluation. After evaluation of the above code,
your resulting node-set will look like this.

```hyperlambda
/* ... rest of code ... */
sort
  person:3
    name:Abraham Lincoln
  person:2
    name:John Doe
  person:1
    name:Thomas Hansen
```

Notice how your supplied callback lambda object is now entirely gone, and only the sorted nodes are left
in your node-tree. Notice also that providing a lambda block to **[sort]** is optional. If you do not provide a
lambda callback, then the default sorting will be used. The default sorting, sorts ascending by
_value of your nodes_. If you wish to sort descending, you can use the alias of **[sort-desc]** which
your **[sort]** Active Event implements as an override.

### Ninja tricks

Below you can find some _"Ninja tricks"_ with P5, helping you achieve some special thing, maybe not
completely intuitive unless explained.

### Parametrizing your expressions

Sometimes you do not know the exact expression during runtime, but want to parametrize your expressions,
according to some argument passed into your code. Imagine something like this for instance, which is
probably a relatively common use-case when consuming P5.

```hyperlambda
_parameter:last
_new-value:Doe
_data
  person
    first:Thomas
    last:Hansen
set:x:/@_data/*/*/{0}?value
  :x:/../*/_parameter?value
  src:x:/@_new-value?value
```

In the above code, we do not know which node beneath our **[person]** is supposed to be updated, but
the actual node is passed in as a parameter through the **[\_parameter]** node's value. By creating a
formatted lazy evaluated expression, which takes the value of our **[\_parameter]** node as an argument -
We can deduct exactly which node beneath **[person]** is supposed to be updated at runtime.

Now of course, in the above example, the **[\_parameter]** node is statically declared, but it doesn't
matter if it is passed into a lambda object by value, or used as a constant, which is shown in the above code.
