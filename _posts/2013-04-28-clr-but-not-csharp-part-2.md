---
layout: post
title: CLR But Not C# (Part 2)
redirect_from:
  - /blog/clr-but-not-csharp-part-2
---

This is a continuation of my [previous article](/blog/clr-but-not-csharp) where I post small things that I notice from an awesome [CLR via C#](http://www.amazon.com/CLR-via-Microsoft-Developer-Reference/dp/0735667454) book. It’s about features (as well as “features”) that are supported by the CLR (and MSIL) but can’t be leveraged when you use C# as a programming language.

## Static Indexers

C# has support for indexers (in Visual Basic .NET they’re called default properties), a special properties that allow you to access object state using square brackets. For example:

<script src="https://gist.github.com/volpav/823ebc039a68b4e46ada.js"></script>

The thing is that you can’t define static indexers in C# (e.g. by marking “this” with “static” access modifier) while CLR as well as Visual Basic.NET are perfectly fine with having static indexers. 

Note: In fact, all object properties (including indexers) are turned into methods with “get_” and “set_” prefixes (for “get” and “set” accessors accordingly) so having a static indexer is just like having a static “get_[indexer_name]” and “set_[indexer_name]” methods which would be a perfectly valid code.

## Custom Indexer Names

Continuing with indexers, C# doesn’t have any language features for defining custom names for indexers. By default the compiler will always emit “Item” as indexer name. In Visual Basic .NET, however, you can easily define a custom name by just giving any valid name to a default property:

<script src="https://gist.github.com/volpav/a505f0c7770c0b5ee45f.js"></script>

To workaround this limitation in C#, you’ll have to use IndexerNameAttribute (and probably also DefaultMemberAttribute for convenience). In fact, String class uses this technique for exposing individual characters via “Chars” indexer.

Note: Some of you might have noticed that when accessing an indexer with the custom name that resides in an assembly that was build using Visual Basic .NET from within C#, the Visual Studio IntelliSense shows you something like “get_[name]” which looks like a method. This, again, demonstrates that properties (including indexers) are nothing more than methods in MSIL.

## Generic Properties

C# doesn’t support generic properties so the following code won’t compile: 

<script src="https://gist.github.com/volpav/99fc09c968fb73679514.js"></script>

The above would probably make sense from the MSIL perspective (because, again, properties are compiled into methods so a generic property would compile into a generic method) but C# team thought that code like this doesn’t make sense conceptually since properties are meant to represent a state of an object rather than its’ behavior.

## “out” And “ref” Parameters

In C# we use “out” keyword to tell the compiler that the specified argument represents a result of the method and so the compiler requires this argument to be initialized within the method. On the other hand the “ref” keyword tells the compiler that the given argument is passed by reference (either a reference to a value for value types or a reference to a pointer for reference types) and so the compiler requires the caller to initialize the argument before passing it to the method. 

CLR doesn’t distinguish between “out” and “ref”.

## “&lt;” And “&gt;” In Type Names

C# doesn’t allow you to use angle brackets as part of the identifier or within the type/method name. But CLR doesn’t have this restriction. Furthermore, C# uses this “feature” internally when generating method names for anonymous methods and lambda expressions (you can see it in the debugger).