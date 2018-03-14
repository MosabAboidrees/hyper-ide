
## p5.data, a memory based database

Phosphorus Five has its own database. This is a memory based persistent lambda database, which is 
extremely fast. p5.data encapsulates that database. It consists of 4 basic Active Events.

* __[insert-data]__ - Inserts new lambda nodes into your database
* __[select-data]__ - Selects existing lambda nodes/values/names from your database
* __[update-data]__ - Updates existing lambda nodes/values/names in your database
* __[delete-data]__ - Deletes existing lambda nodes from your database

Together they create a tree-based, persistent, still memory based storage for your lambda objects.
This means that you can put both _"data"_ and _"code"_ in it. It is relatively fast (obviously, since
it is memory based) - But the flipside, is that it does not scale beyond whatever amount of memory you
have in the machine that's running your Phosphorus Five installation. Hence, you should only use it for
smaller datasets, and not big datasets containing thousands of records.

### Inserting data [insert-data]

To insert items into your database, you can use for instance a piece of lambda, such as shown below.

```hyperlambda
insert-data
  foo.my-object
    name:John Doe
    address:Foobar Rd. 64
```

The above code will insert a new item into your database, and automatically assign a new unique
ID (Guid) to it, which uniquely identifies it within your database. The ID for your item, is returned
as the value of the item you insert, after the invocation to **[insert-data]**.

To select the item you inserted, you can use for instance the following code.

```hyperlambda
select-data:x:/*/*/foo.my-object
```

After you evaluate the above **[select-data]** lambda, assuming you previously evaluated the
above *[insert-data]* to insert the item - You will end up with something resembling the following.

```hyperlambda
select-data
  foo.my-object:guid:9a44df94-549c-49ba-b8a1-3f6d62a9c835
    name:John Doe
    address:Foobar Rd. 64
```

The ID (the ":guid:xxx" parts) will of course be different for your system.

Internally, the database creates its own file-structure, which by default should exist within your
_"/db/"_ folder on disc. Though the exact folder, can be overridden in your web.config. If you open
your db folder, you will see that it contains one or more folders, named "/db0/", "/db1/", etc. These
folders again, contains one or more files, named "db0.hl", "db1.hl", etc. By default, your database
will serialize maximum 32 objects into each file, and a maximum of 256 files into each folder. So
as your database grows, it will start creating more and more folders and files. This makes it very
fast to serialise items, since it allows the implementation to serialise only a small fraction of
its content, as updates and changes occur. p5.data is thread safe.

Each file is in fact a simple Hyperlambda file, and can (in theory) be edited with Notepad or
TextEdit if you wish. This simple trait of p5.data, makes it extremely easily understood, robust,
and easy to fix, if corruption should occur somehow. However, as a general rule, you should not edit
these files yourself, unless you stop your web server first, before you start editing them. Let
p5.data take care of its own file structure, unless you really have to edit them yourself.

Your disc will only be touched during inserts, deletes and updates. During **[select-data]**, which
normally should be the bulk of your database invocations, it never accesses your disc in any ways.
Which makes it extremely fast for select data operations. In addition, due to the underlaying file
structure, using multiple files for its data, it rarely have to update (save) more than a fraction
of your database files, depending upon how many data records you change in it.

This makes it extremely fast for small datasets, while not very scalable for huge documents,
images (although possible), other binary data, or huge datasets. Use it with _care_. Phosphorus Five
contains a MySQL plugin, which you can use for data that requires more space than that which is
feasible to expect for p5.data.

### Inserting multiple items at once

The database will create a "lock" on itself during insert operations. This makes it easy to insert
multiple items at once, without having race conditions, and similar types of problems during insertion.
To insert multiple items, simply add up more than one child node as children of **[insert-data]**,
as illustrated below.

```hyperlambda
insert-data
  foo.my-type
    name:John Doe
  foo.my-other-type
    address:Foobar Str. 45
  foo.some-third-type
    customer:foo-bar
      name:Thomas Hansen
      address:Unknown
      email:mr.gaia@gaiasoul.com
```

The above **[insert-data]** invocation, will insert three items, of different "types", into your database,
using one single "lock" operation. If you watch its result, after evaluating it, you will see that each
item has its automatically generated ID returned, and nothing else.

The objects you insert into your database, can be as deep and complex as you wish, containing any types
Phosphorus Five supports. You can also insert as many items as you wish, in one go. And in fact, the
above **[insert-data]** invocation, will only create one lock, so if you can, you should insert all
"related items" at once, to get away with as few database locks as you can.

If you wish, you can also supply an explicit ID to **[insert-data]**, by adding it as the value of each
node inserted. Your ID can also be any type P5 supports. However, the ID must be unique for your server.
Below is an example of inserting items with an explicit ID.

```hyperlambda
insert-data
  foo.my-item-type:some-ID-goes-here
    name:John Doe
  foo.my-item-type:some-OTHER-ID-goes-here
    name:Jane Smith
```

### [select-data] dissected

To understand **[select-data]**, is to understand the internal structure for how items are stored in memory.
Your database has one root node, in addition to several file nodes. These nodes _should_ be ignored, which is
why all expressions starts out with two children iterators (`:x:/*/*`). Internally, the database is simply a
huge graph object, or tree structure, with all items from your p5.data files, internally stored within in
as graph objects. The selectors above hence, are lambda expressions, referencing one or more items inside
of this tree structure. Lambda expressions are kind of Phosphorus' version of XPath.

Notice also, that expressions supplied to the database Active Events, does not use the database Active Event
invocation nodes, as their identity node, but rather the database root node. After the two initial children
iterators, you can supply any combination of iterators you wish. For instance, to select two of the three
items we inserted above, you could use something like this.

```hyperlambda
select-data:x:/*/*/foo.my-type|/*/*/foo.some-third-type
```

The above code, assumes some basic knowledge about boolean algebraic expressions. But basically, it selects
all items having the name of _"foo.my-type"_ and all items having the name of _"foo.some-third-type"_, ignoring
everything else. Assuming you evaluated the insert-data example that inserted three items at once above,
your result will look something like this.

```hyperlambda
select-data
  foo.my-type:guid:9eb516b4-8a6e-4d8c-850d-834f2f184c24
    name:John Doe
  foo.some-third-type:guid:ae3be54a-113a-4c6e-8031-71639a093499
    customer:foo-bar
      name:Thomas Hansen
      address:Unknown
      email:mr.gaia@gaiasoul.com
```

You can (of course) use any expressions in p5.data as you can legally use in any other parts of P5, as long as
you do not select further upwards than at least "two children nodes from root". Meaning, starting your expressions
with for instance `:x:/*/*/... plus other iterators ...`.

You can also select items from your database according to their IDs. However, since the automatically generated
IDs are Guids, you need to type them as such in your value iterators. Below is an example for you.

```hyperlambda
// Exchange the Guid below, with the Guid that was generated for one of your items
select-data:x:"/*/*/=:guid:9eb516b4-8a6e-4d8c-850d-834f2f184c24"
```

Notice how I need to put my entire expression into a string literal. This is because my expression contains a
colon (:), which is a special character in Hyperlambda, declaring where the name and/or type ends, and the
value starts.

With **[select-data]**, you can also count items in your database. Consider this code for instance.

```hyperlambda
select-data:x:/*/*/~foo.?count
```

Which will return the number of items in your database, having a root node, containing the string "foo.",
somewhere within its name.

### Deleting data with [delete-data]

The **[delete-data]** Active Event, takes similar types of expressions as **[select-data]**. However,
instead of selecting items, it deletes items from your database. To delete one of the items we inserted
above, you could use something like this.

```hyperlambda
delete-data:x:/*/*/foo.some-third-type
```

If you evaluate the above p5.lambda, you will see that it returns the number of items that was actually
deleted for us. Which for the above lambda, should be no more than one item of course. You can also of
course delete items by their IDs, or use any other legal expressions, with boolean algebraic expressions,
and any amount of complexity you choose. You can also delete multiple items using **[delete-data]**,
which of course will only create one lock on your database for you. Whatever lambda expressions you can
legally create, you can also use to delete data using **[delete-data]**. Example given below.

```hyperlambda
delete-data:x:/*/*/~foo.
```

You can also delete only parts of your graph objects, such as for instance.

```hyperlambda
delete-data:x:/*/*/~foo./*/name
```

Which will delete all **[name]** children nodes of objects having _"foo"_ in their type declaration. In general,
you can modify any parts of your database graph object you want to, using p5.data.

### Updating data with [update-data]

The **[update-data]** Active Event is probably the most complex of all data events. This is because it
allows to update any parts of your tree, any ways you see fit, including values of nodes, names of nodes,
and even the nodes themselves. It takes a **[src]** argument, which can be replaced with any Active Event
invocation you wish. Let's first take a look at a traditional **[src]** invocation.

```hyperlambda
// Inserting item to have something to update
insert-data
  foo.test-update
    name:John Doe

// This is our update invocation
update-data:x:/*/*/foo.test-update/*/name?value
  src:Jane Smith

// Selecting item to verify it was actually updated
select-data:x:/*/*/foo.test-update
```

In the above lambda, we first insert an item, with a name node, having the value of _"John Doe"_.
Afterwards, we update __only__ the **[name]**, of our inserted items, such that it becomes _"Jane Smith"_.
To show our change, we invoke **[select-data]**, at the end of our lambda.

**Warning**, if you had multiple items in your database, with the root node's name being **[foo.test-update]**,
then all items would be updated with the above **[update-data]** invocation.

To update the entire item, we could use something like this.

```hyperlambda
update-data:x:/*/*/foo.test-update
  src
    foo.test-update
      first-name:Thomas
      surname:Hansen
select-data:x:/*/*/foo.test-update
```

Or, if you wish to ensure that only one item is updated, you could pass in its ID as a part of
your expression, as is shown below.

```hyperlambda
update-data:x:"/*/*/=:guid:f89e6955-8dee-41ec-aa05-e8f2e450c073"
  src
    foo.test-update
      first-name:Peter
      surname:Smith
select-data:x:/*/*/foo.test-update
```

The above code, uses the automatically generated ID for the item from my database. The guid needs
to be replaced with the guid generated by your system, for one of your items, if you wish to test
this for yourself. If an exception occurs, during both insert, delete and update, then the entire
batch of change will be rejected. Transactions are **implicit** in p5.data.
