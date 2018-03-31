---
layout: post
title: CLR But Not C#
redirect_from:
  - /blog/clr-but-not-csharp
---

Among the books that I’m currently reading right now is Jeffrey Richter’s [CLR via C#](http://www.amazon.com/CLR-via-Microsoft-Developer-Reference/dp/0735667454) (I should have read this book long time ago but never really managed to allocate enough time for it). Apart from being an very interesting read covering the internals of the Common Language Runtime and the ways different components (for example, the gargabe collector) work, this book also covers some of the CLR “hidden” features that are not supported by any language (except of MSIL). So I thought I’ll just post some of the points that I noted and that might be interesting to others. I’ll be giving all examples in C# as well.

## Static Constructors In Interfaces

As you know, you can’t make a constructor be a part of the interface meaning that the following code won’t compile:

<script src="https://gist.github.com/volpav/5332226.js"></script>

It doesn’t matter if it’s a default (parameterless) constructor or the one with parameters - you simply can’t declare constructors on interfaces. But what about static constructors (the ones that are called before any instance of the class is created and before any static method is referenced)? Well, it turns out that although C# doesn’t allow static constructors on interfaces either, CLR is perfectly fine with that. It’s just something that C# team decided not to support by their compiler.

## Methods That Differ Only By Return Value

Consider the following example:

<script src="https://gist.github.com/volpav/c74caa6f9d68b130ac59.js"></script>

This code won’t compile because the above two methods differ only my return value. But guess what: CLR does support this feature and there’s no disambiguation when you look at the corresponding MSIL code. In fact, C# also leverages this feature internally when you use conversion operators:

<script src="https://gist.github.com/volpav/b9a97596b49326210d57.js"></script>

## Value Types With Default Constructors

This one is quick: you can’t declare a default (parameterless) constructor on a value type (struct) in C# but you can easily do it in MSIL.

## call vs. callvirt

For the purpose of this article, I’m not going to explain in details on how CLR “call” instruction differs from “callvirt” (in short, “callvirt” allows calling an instance method virtually whereas “call” - non-virtually). What is more important for us is that “callvirt” makes a check whether the object instance that the method is being called on is not null (and throws an exception if it is). On the other hand, “call” doesn’t make this check so basically you can call instance methods on a null object:

<script src="https://gist.github.com/volpav/2d83784f46c7006f7187.js"></script>

Well, of course C# will throw a NullReferenceException when the last line is being executed. This is because C# compiler almost always emmits “callvirt” instead of “call” instruction (“call” is only used when you do “base.Something()”). There’re at least two reasons for this:

- Make errors in the code be discovered sooner (it’s a rare case when instance methods don’t access object state so it makes perfect sense to always check that the object is not null before calling an instance method on it).
- With “call” there’s a chance of introducing a breaking change: let’s say that you made a public API with the class that have a normal method. When the consumer references the assembly with your API, the C# compiler emmits the “call” instruction to call the method. Later you decide that this method should be overridable and mark it with “virtual” keyword. Seems pretty safe but now the compiler needs a “callvirt” instruction for calling this method (and therefore, finding out which implementation to call) but if the consumer doesn’t recompile his code with the new version of your API, his application can start behaving really strange because there’s still a “call” instruction and not the “callvirt”.

## Less Restrictive Overrides

Consider the following example:

<script src="https://gist.github.com/volpav/651728d938e8ea31231e.js"></script>

Notice how the access modifiers for “TestMethod” are changed in derived classes. The code for the first two classes (“BaseClass” and “Derived1”) compiles just fine whereas the code for the third class (“Derived2”) won’t compile because C# allows you to only make overrides more restrictive (protected -> private) and not less restrictive (protected -> public). CLR doesn’t carry this constraint so the corresponding MSIL code for the “Derived2” class would execute just fine.

## “Family And Assembly” Methods

The last example is about accessibility of the methods. In C# you can have methods that are:

- Public (“public” access modifier).
- Private (“private” access modifier).
- Family (“protected” access modifier).
- Assembly (“internal” access modifier).
- Family or assembly (“protected internal” access modifier).

CLR also supports “Family and assembly” access type. This means that the method is accessible from within the declaring type, nested and derived types but only if they’re declared in the same assembly. Well, apparently C# team didn’t think of this as a very useful feature so it’s not supported in this language.

That was all for now. I will post some more interesting findings as I collect them.