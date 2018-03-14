
## p5.http - HTTP REST events

This component contains the Active Events necessary to create HTTP REST requests. It contains full support for
all four basic HTTP verbs.

* __[p5.http.get]__
* __[p5.http.post]__
* __[p5.http.put]__
* __[p5.http.delete]__

To retrieve a document, using HTTP GET for instance, you could do something like the following.

```hyperlambda
p5.http.get:"https://google.com"
```

The above will return something similar to this.

```hyperlambda
p5.http.get
  result:"http://google.com/"
    status:OK
    Status-Description:OK
    Content-Type:text/html; charset=ISO-8859-1
    Last-Modified:date:"2016-11-16T13:44:57.581"

    /* ... the rest of the HTTP headers returned from google.com ... */

    content:@"... document from google.com ..."
```

The actual document, will be returned as **[content]**, inside the above **[result]** node. While the HTTP headers
returned, can be found just beneath the **[result]** node. If you wish, you can retrieve multiple documents at the
same time, by supplying an expression to your invocation. An example of the latter is given below.

```hyperlambda
_urls
  url1:"http://google.com"
  url2:"http://digg.com"
p5.http.get:x:/-/*?value
```

Notice, this will create your requests sequentially, and not in parallel. If you wish to create parallel requests,
you'll have to dive into the p5.threading project of P5. You will still end up with one **[result]** node above,
for each URL you supply to it. You can also supply any HTTP headers you wish, as illustrated below.

```hyperlambda
p5.http.get:"https://httpbin.org/get"
  Foo-Bar:Some data goes here
src:x:/../**/content?value.string
```

p5.http handles every argument to its events that are not **[result]**, **[.onrequest]**, **[.onresponse]** and
**[content]** as an HTTP header.

There are 4 basic Active Events in this project.

* __[p5.http.get]__ - HTTP GET - Returns document
* __[p5.http.post]__ - HTTP POST - Posts data
* __[p5.http.put]__ - HTTP PUT - Puts data
* __[p5.http.delete]__ - HTTP DELETE - Deletes data

### POST and PUT

The POST and PUT events, automatically recognize Hyperlambda, and allows you to transmit a lambda node structure,
without first converting it into text. To transmit a piece of Hyperlambda to another server, you could use
something like the following.

```hyperlambda
p5.http.post:"https://httpbin.org/post"
  content
    _data
      no1:Thomas
      no2:John
src:x:/../**/content?value.string
```

If you wish to POST or PUT simple content, you can do such a thing with something resembling the following.

```hyperlambda
p5.http.post:"https://httpbin.org/post"
  content:foo bar
src:x:/../**/content?value.string
```

### POST'ing and PUT'ing files

If you have a big file you wish to POST or PUT, you can achieve it using the following syntax.

```hyperlambda
p5.http.post:"https://httpbin.org/post"
  .onrequest
    .p5.io.file.serialize-to-stream:/application-startup.hl
src:x:/../**/content?value.string
```

Although I am not sure if there really exists many valid use-cases for this, you could provide a path to multiple
paths if you wish, to serialize multiple files into your HTTP request stream in one go. And example is given below.

```hyperlambda
_files
  /application-startup.hl
  /system42/application-startup.hl
p5.http.post:"https://httpbin.org/post"
  .onrequest
    .p5.io.file.serialize-to-stream:x:/@_files/*?name
src:x:/../**/content?value.string
```

Exchange the above invocation to **[p5.http.put]** if you wish to use PUT the file instead. However, a more useful
alternative to the above, would probably be to use p5.mime to create a MIME message, which you transmit to some
HTTP endpoint instead. See documentation further down in this document for an example. The above _"post"_ invocation,
will not read the files into memory, before they're transmitted to your HTTP endpoint. But rather, copy the stream
directly from disc to the request stream. This allows you to transfer huge files, without exhausting your server's
resources. This will use a plugin from p5.io, which takes a stream as an argument, allowing you to copy a file
directly into the HTTP request stream, instead of loading the file into memory first. This is highly useful when
you want to transfer large files, without exhausting your server's memory or resources in any ways.

Notice how we wrapped our invocation to **[.p5.io.file.serialize-to-stream]** inside of another node called
**[.onrequest]**, to inform the p5.http component that it should use our custom serialization logic, instead
of looking for a **[content]** node.

### MIME and PGP cryptography support for POST and PUT

Instead of supplying a **[.p5.io.file.serialize-to-stream]** argument for your POST and PUT operations, you
can alternatively have a MIME message automatically created for you, using the automatic plugin into p5.mime,
and POST or PUT a MIME message. If you wish to use this feature, you would instead of supplying a
**[.p5.io.file.serialize-to-stream]** argument, supply a **[.p5.mime.serialize-to-stream]** node, containing
one or more of the MIME types supported by p5.mime. Below is an example of creating a multipart/mixed MIME
message, with two text MIME nodes.

```hyperlambda
p5.http.post:"https://httpbin.org/post"
  .onrequest
    .p5.mime.serialize-to-stream
      multipart:mixed
        text:plain
          content:Foo bar
        text:html
          content:<p>Foo bar</p>
src:x:/../**/content?value.string
```

You can only provide one MIME message to the **[.p5.mime.serialize-to-stream]** Active Event, but this MIME
entity can be a multipart, containing several leaf entities. The above construct, allows you to use the full
feature set from p5.mime, which among other things allows you to create PGP encrypted and cryptographically
signed HTTP MIME requests, such as the following is an example of.

```hyperlambda
/*
 * Checking if our dummy PGP key exists from before, and if not, creating it.
 */
p5.crypto.list-public-keys:SOME_DUMMY_PGP_KEYPAIR@SOMEWHERE.COM
if:x:/-/*
  not
  p5.crypto.create-pgp-keypair
    identity:John Doe <SOME_DUMMY_PGP_KEYPAIR@SOMEWHERE.COM>
    strength:1024
    password:foo

/*
 * Creating our POST request, making sure we encrypt and sign it with the above PGP key.
 */
p5.http.post:"https://httpbin.org/post"
  .onrequest
    .p5.mime.serialize-to-stream
      multipart:mixed
        encrypt
          email:SOME_DUMMY_PGP_KEYPAIR@SOMEWHERE.COM
        sign
          email:SOME_DUMMY_PGP_KEYPAIR@SOMEWHERE.COM
            password:foo
        text:plain
          content:Foo bar
        text:html
          content:<p>Foo bar</p>
src:x:/../**/content?value.string
```

This is a pretty kick ass cool feature, allowing you to create PGP encrypted web services, in addition to
cryptographically sign your web service invocations. Yet again, the MIME message is serialized directly into
the stream, and never loaded into memory, which means you can create humongously large HTTP MIME REST requests,
without exhausting your server's memory. You can also create your own serialization plugins, using the
**[.p5.mime.serialize-to-stream]** event as an example. Notice that all plugin serializer having supplied
the **[.onrequest]** must have exactly one child node, having the name of the Active Event you wish to use
for serializing your request.

### Rolling your own serializer in C#

The above logic also allows you to create your own C# plugins entirely from scratch, using the above construct
to serialize into the HTTP Request stream your content one way or another. This is because the above
**[.p5.mime.serialize-to-stream]** node and **[.p5.io.file.serialize-to-stream]**, becomes an Active Event
invocation, that passes in the HTTP request stream of the HTTP request and the entire **[.p5.mime.serialize-to-stream]**
node directly as its parameters to the event.

If you wish to use the above construct to create your own plugin, realise that the arguments passed into your
own Active Events becomes a `Tuple<object, Stream>`, where you can find the old value of your `e.Args.Value`
as the object in Item1, and the HTTP request stream as Item2. This is necessary to support Active Events that
contains values in their main `Node` argument. This feature allows you to create your own serialization logic
for the p5.http component, serializing any types of content you wish during your requests from C#. Notice,
if you do, you do not gain ownership of the HTTP request stream, and you should hence not close it or dispose
of it any ways, but simply serialize into it, and let it pass out of your Active Event, being still active and
open.

### GET'ing files

If you instead want to retrieve a document using HTTP GET, and save it directly to disc, without loading it into
memory, you can use an **[.onresponse]** argument, similarly like you used an **[.onrequest]** argument in the
above examples, and pass in the **[.p5.io.file.save-to-stream]** Active Event. Consider the following code.

```hyperlambda
p5.http.get:"https://google.com"
  .onresponse
    .p5.io.file.save-to-stream:/foo-google.txt
```

The above code will download google.com's index document, and save it to your private documents folder. It will
still return all HTTP headers from the server, but no **[result]** node will exist after invocation.

### Ninja tricks (Hyperlambda Web Services)

Due to the extreme dynamic nature of Hyperlambda, you can easily transmit Hyperlambda, over for instance an
HTTP POST request, to have it evaluated on another server, and then return its result to the caller as Hyperlambda.
Consider creating the following Hypereval page, that reads the body of your request, and evaluates it as Hyperlambda,
for then to return the result to caller. To create such a web service endpoint, you could create something like
the following.

```hyperlambda
// Retrieves the HTTP POST request body.
p5.web.request.get-body

// Evaluates the request body as Hyperlambda.
eval:x:/@p5.web.request.get-body

// Converts the return value from the evaluated Hyperlambda to Hyperlambda code.
lambda2hyper:x:/@eval/*

// Making sure we return it with the correct Content-Type, such that
// invoker of Web Service can more easily recognize it as Hyperlambda.
p5.web.header.set
  Content-Type:application/x-hyperlambda

// Returning the Hyperlambda to caller.
p5.web.echo:x:/@lambda2hyper?value
```

Then evaluate the following code, assuming your web server is listening on port 8080.

```hyperlambda
p5.http.post:"http://localhost:8080/url-to-above-page"
  Content-Type:application/x-hyperlambda
  content
    _data
      no1:Thomas
      no2:John
    for-each:x:/-/*?value
      eval-x:x:/+/*/*/*
      add:x:/../*/return
        src
          p5.web.widgets.create-literal
            element:h3
            innerValue:x:/@_dp?value
    return
hyper2lambda:x:/../**/content?value.string
eval:x:/-
```

After evaluating the above HTTP POST request, you should have two additional widget on your page,
and the return value should look something like the following.

```hyperlambda
p5.http.post
  result:"http://localhost:1176/url-to-above-page"
    status:OK
    Status-Description:OK
    Content-Type:application/x-hyperlambda; charset=utf-8

    /* ... more HTTP headers ... */

    content
      p5.web.widgets.create-literal
        element:h3
        innerValue:Thomas
      p5.web.widgets.create-literal
        element:h3
        innerValue:John
eval
```

To understand the beauty of the above construct, realize that what we actually did, was to transmit Hyperlambda
to another server, for then to have that server evaluate the Hyperlambda, allowing the caller's code to decide
what to evaluate on the web service end-point, and what to return after evaluation. In theory, this makes it
possible for you to create _one single Web Service endpoint_, for every single Web Service needs you can
possibly have.

### Warning!!

The above construct, allows anyone to evaluate any piece of Hyperlambda on your server, which of course is
an extremely dangerous security issue, effectively opening up your server entirely for any arbitrary piece
of code anyone wants to evaluate on it. If you combine the above construct, with the PGP cryptography from
the p5.mime project, you can require that the client invoking your web service is trusted, by only allowing
requests from a list of pre-declared trustees, and require clients to cryptographically sign their MIME
messages in a PGP MIME multipart message. Still, you would need to be 100% confident in that the client's
private PGP key has not somehow been compromised, and that you can trust the client supplying the Hyperlambda.

There are ways to further refine this, and increase the security, by requiring the client to only supply
a sub-set of Active Events, through using e.g. the **[eval-whitelist]** Active Event from p5.lambda, when
evaluating the incoming and returned lambda. Please see the **[eval-whitelist]** Active Event
for details about this.

If you setup such a web service end-point correctly, at least in theory, your web service should be 100%
safe from intrusion and malicious code - Yet still, you would have a lambda web service, allowing the
client to supply the code to evaluate for you.
