---
layout: post
title: Using Fluent Interfaces For Exception Management
redirect_from:
  - /blog/using-fluent-interfaces-for-exception-management
---

As part of my latest project I’m writing a recursive descent parser. Obviously, the parser will sometimes fail to process the particular expression due to its invalidity (empty expression, malformed constructs, etc.). Since I’m writing all this in C#, I decided to reports about errors by throwing exceptions. Turned out, for my case I needed a bit special approach in constructing, initializing and throwing exceptions other than just a simple “throw new ...”. Let’s take a look at what I came up with.

## The Expected Behavior

Let me elaborate on how I wanted to manage exceptions for the above mentioned task. There’re many different types of exceptions that the parser can throw. Here’s a short list:

- SyntaxException
- UnexpectedTokenException
- UnexpectedEndOfStreamException
- ...

All the above types are ultimately derived from “ParseException” (which, in turn, is derived from “System.Exception”). So for different kinds of errors I’m going to create an instance of the corresponding exception type, fill out some properties and throw it. Something like this:

<script src="https://gist.github.com/volpav/860893654e4c5ac808df.js"></script>

Notice that I’m filling out three properties before throwing an exception. One of the goals is to pass as much related information as possible. This will help me debugging the parser and will also make my unit tests more “harsh” minimizing the risk of unintended changes to the behavior of the parser. Since different exception types have different set of properties (that must be initialized before throwing an exception), the code for throwing these exceptions will be different and copy-pasting these snippets all over the parser implementation is definitely not the way to go.

Another characteristic that I wanted my exceptions to have is maximum immutability. In other words, all properties must be read-only and initialized through the constructor. The reason is that I don’t want the code handling the exception to “lie” to the code on the upper level about what has actually happened by modifying the data supplied with the exception (and re-throwing it eventually). When I think about this, it reminds me of server-side events in ASP.NET where the majority of arguments passed with an event are immutable. There it makes even more sense because you can’t be sure in which order the event handlers are called and so you don’t want to mess around with the event data within the particular handler. So, taking into consideration the “immutability” constraint the code will now look like the following:

<script src="https://gist.github.com/volpav/47fa43f8160365a561f2.js"></script>

Lastly, I’d like to wrap the above constructs into a method to make the code more clear and more maintainable. Something like this:

<script src="https://gist.github.com/volpav/7a651faaeed6da00f6ec.js"></script>

This doesn’t look very pretty, does it? It can be changed to something like the following:

<script src="https://gist.github.com/volpav/884396d153eab083e06a.js"></script>

Okay, the signature of the method was simplified but basically nothing was gained other than having a central place for “throw” statement. And it still looks pretty ugly.

## Building a Fluent Interface

As per Wikipedia article, a fluent interface is an implementation of an object oriented API that aims to provide for more readable code. A fluent interface is normally implemented by using method chaining to relay the instruction context of a subsequent call and the context is:

- Defined through the return value of a called method;
- Self-referential, where the new context is equivalent of the last one;
- Terminated through the return of the void context.

In this particular case, I’m going to use extension methods and some Dynamic LINQ for implementing the fluent interface over exception handling. I’ll start by defining the beginning of the method chain:

<script src="https://gist.github.com/volpav/3bb128cc13a4b6801974.js"></script>

The “ExceptionContext” class looks like the following:

<script src="https://gist.github.com/volpav/a0bea49670cf91236631.js"></script>

So, as you can see the exception context contains some useful information about the expression itself. This information is passed through the chain to make initialization of the newly constructed exception much easier:

<script src="https://gist.github.com/volpav/691696681104765d58e3.js"></script>

One thing to notice about the previous code snippet: I’m using Reflection in order to update the value of the backing field for the “Message” property of the given exception instance. This is because the “Message” property is read-only. Although this works, it’s kind of a hack because you never know when BCL team is going to rename/remove the “_message” field or change the implementation of the “Message” property getter so it won’t use this field (in other words, we don’t have control over the backing field so we should at least handle the possible error somehow).

Next there’s the initialization of exception properties. Taking into consideration the immutability rule that I talked about earlier and since the exception instance is created using the parameterless constructor, I’m going to use Reflection again in order to set property values (it’s much less of a hack now since I'will be examining with my own types which I have full control over). Here’s the first method which is called “Initialize”:

<script src="https://gist.github.com/volpav/a63a6d5bc97a0fbd549d.js"></script>

In the above code snippet I’m using Dynamic LINQ to provide a nice, clear syntax of defining which property the caller wants to initialize. Also, as you might have noticed, the method accepts exception-related context and returns a member-related context which looks like the following:

<script src="https://gist.github.com/volpav/3a47c780cbf380d3b398.js"></script>

And here’s the method that sets the value of the property specified earlier in the chain by “Initialize”:

<script src="https://gist.github.com/volpav/f3dbd2c50c5b8d1b1f45.js"></script>

Again, the Reflection is used here to set the value of a read-only property (the property itself is defined as “[Property_name] { get; private set; }”). At the end, the parent (exception-related) context is returned allowing the caller to keep joining “Initialize+With” pairs or any other methods defined on the given context.

The last method in the chain is “Throw” which basically throws an exception that just went through the chain:

<script src="https://gist.github.com/volpav/8ab92949d3461822c0f0.js"></script>

Nothing fancy about the last method, it just throws an exception (although my implementation performs some additional tasks before throwing the exception).

## Usage

Below are some of the examples of using the fluent interface we just wrote:

<script src="https://gist.github.com/volpav/0978e98a7d3121b9571c.js"></script>

Now with this skeleton we can easily add more functionality (for example, logging) without sacrificing the readability.

## Conclusion

Fluent interfaces is a great way of making your code easier to read and understand and .NET (and C# in particular) provides a very convenient set of features to make building and consuming such interfaces pretty much a walk in the park.