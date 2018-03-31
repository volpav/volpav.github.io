---
layout: post
title: Re-Discovering Microsoft Web API
redirect_from:
  - /blog/re-discovering-microsoft-web-api
---

Many applications and services these days are being implemented with the use of Web APIs. Some of these products provide publicly facing API endpoints (Facebook Platform), others utilize service oriented architecture internally (Amazon) and of course, there are those that combine both techniques (Azure). Before building a Web API, you need to consider the possible use cases and so your design and implementation must reflect (and accommodate to) those. In this article I’m going to cover some of the not-that-widely-used but extremely useful (in certain cases) features and patterns that can be leveraged in ASP.NET Web API framework.

Since this article is somewhat big, here’s a table of contents:

- Asynchronous actions
- OData
- Batching
- CSRF prevention
- Request throttling
- Long-running tasks and progress
- Conclusion

So, without any further delays, let’s begin!

## Asynchronous Actions

Many people often get confused whether they should make their Web API actions asynchronous by returning “Task<T>” along with marking the action method with “async” keyword. Generally speaking, the use of asynchronous actions is beneficial when the work you’re doing inside the action method is either network-bound (e.g. calling a web services) or I/O-bound (e.g. reading a large file from disk). Also, you can dramatically improve the responsiveness and resource utilization if you run a number of such work items in parallel (e.g. by using “Task.WhenAll”). With CPU-bound work (e.g. processing the large in-memory structure using LINQ), there’s no benefit of making your actions asynchronous. In fact, there is even a slight overhead that involves the use of task scheduler and context switches.

Here’s a typical example of asynchronous Web API action method:

<script src="https://gist.github.com/volpav/dfa184db4d47b418b625.js"></script>

Notice the use of “GetStringAsync” in conjunction with “await” keyword. This will allow the current thread to be released back to the thread pool without being blocked by the inner web request. When the response arrives, another thread pool thread is going to pick up the work which is parsing the output and returning the list of blog posts.

## OData

Open Data protocol (OData) is a standard for providing CRUD (Create Read Update Delete) access to the data via web services. OData is an open standard which was initially developed (and which is being actively maintained) by Microsoft. OData adds a ton of useful functionality to your web endpoints and really empowers your data in many ways.

OData is a huge topic and without going into much detail, I’d like to give you a quick overview of the support for this technology in Web API. But first, a couple of words about the use cases, namely when OData can be extremely useful as well as when you probably want to avoid it. Personally, I think OData makes perfect sense as a higher level infrastructure DAL (Data Access Layer) in a SOA-based scenarios. Let’s say you have a bunch of internal data services that communicate with each other. In this case, in terms of data processing, you’d probably want to provide as much flexibility with these services as possible (e.g. service discovery and proxy code generation, LINQ over web service, etc.). On the other hand, when designing a public API for third parties, unless you have a lot of compute power under your belt, you’d probably want to limit the number of operations available on your data to a reasonable sub-set. Otherwise, it might be hard to scale such a web service (e.g. clients performing rich queries using “$filter”, “$expand” and/or “$orderby” which can result in a heavy load on your data warehouse because the actual data is not optimized for such queries). So, when designing a data service, you need to choose what makes most sense considering the purpose of the service plus the possible use cases.

Let’s now take a look at the example of a simple OData service as well as the code for consuming this service. For those using Visual Studio 2013, the good news is that OData scaffolding has been greatly improved in this version of the IDE. I’m using Visual Studio 2012 and the first thing I do is I create an MVC4 project using an “Empty” project template. The next thing is to add Web API 2 OData support using NuGet (just search online for “OData” in the package manager). Make sure you uninstall existing Web API packages first and install Web API 2 bits instead (along with OData support mentioned earlier).

Now, let’s define the data we’re going to expose over the web service:

<script src="https://gist.github.com/volpav/5cb8bf0238ddb6891e82.js"></script>

After we’ve defined our data entities, we now need to define an EDM model that will expose the definition of our data to the clients:

<script src="https://gist.github.com/volpav/b0f29f22c1e8c151f31e.js"></script>

In the example above I’m also defining OData route for the newly created EDM model using “MapODataRoute” extension method. Finally, here’s our controller class that connects our data with operations that are available on it:

<script src="https://gist.github.com/volpav/55a960703dcbbb1d824e.js"></script>

Notice the use of “QueryableAttribute” as well as the return type of “IQueryable<BlogPost>”. These two will enable rich OData queries over the collection of blog posts.

Now when the service is implemented, let’s try consuming it. For this purpose I created a simple console application (File -> New Project -> Visual C# -> Console Application). As I mentioned earlier, one of the cool things about OData services is that they’re fully discoverable (via the “#metadata” suffix), so to make our life easier, we’re going to add a service reference (via “Add Service Reference…” dialog) to our data service. In my case, the URL of the service is “http://localhost:51743/odata/” (for the simplicity, I chose to host this example using IIS Express).

After the service reference has been added, here’s how easy it is to consume the data (I’ve told Visual Studio to put service proxies into the “BlogPostsService” namespace):

<script src="https://gist.github.com/volpav/05f64fb7398fccbea223.js"></script>

Notice how we’re using LINQ over web services which is super nice.

## Batching

I’m really excited about the new batch support that just came out with the release of Web API 2 OData. Batching is a way to pack several different API requests into the single HTTP POST request. The framework middleware then unpacks the request and re-routs the individual requests to the appropriate API methods. Similarly, multiple responses are then going to be packed together and sent back to the client as a single HTTP response. This feature allows you to decrease the traffic between the server and the client dramatically making the entire communication much less “chatty” (and therefore, making your API more scalable).

Let’s take a look at how to integrate batching support into your API. As an example, I’ve created an MVC4 project (again, using the “Empty” project template). As with the previous example, you’ll need to uninstall existing Web API package and install Web API 2 instead (no need to install OData support since we’re not going to use it here).

The first thing we need to do is we need to register a special route that we can forward all our batches to. Here’s how it’s done:

<script src="https://gist.github.com/volpav/82fdb6286f9d7fad9902.js"></script>

Next step is to add an API controller. We’re going to use the domain model with blog posts from the previous example and the controller itself is looking extremely simple in this case:

<script src="https://gist.github.com/volpav/8a55f1719cb1af1a0a3f"></script>

(don’t be confused with the new “Add” method on the repository class. This method simply adds a new object to the underlying collection and returns that object).

As you can see, our API controller is totally unaware of the batch handler registered earlier. This is, of course, logical: there should be absolutely no difference from the API point of view between performing a batch and a normal (single) request. The batching is handled on the transport level down the stack.

Now, to demonstrate the use of batching I’ve created a tiny JavaScript library (the source code of the library can be found here: https://github.com/volpav/batchjs). Here’s how the main view looks like:

<script src="https://gist.github.com/volpav/536028155b0274f893d9.js"></script>

The central part of the view is the call to the “ajaxBatch” method. The semantics of this method are more or less the same as with the native jQuery “ajax” method, although there’s only one actual HTTP request that is being made and the batch contents is specified by defining the “data” array. Notice that there’s also an additional argument “data” which is passed to “complete” handler. The argument contains an array of object, where each element of the array corresponds to a single API response within the batch (the library assumes that the order of the responses within the batch is the same as the order of the requests, although this can be customized at the server level). Each element represents an object with two fields: “status” (HTTP status code, e.g. 200) and “data” (evaluated response data, if any).

## CSRF Prevention

Cross-site request forgery (CSRF) is a type of a website (web service) vulnerability when unauthorized calls to the service are made through the malicious “man in the middle” software/person. These types of attacks are very popular because they’re easy to perform and because there’re many developers who violate the concepts of HTTP and neglect even the basic protection. This is, of course, much more relevant when dealing with public APIs rather than the ones that are designed for (and exposed to) internal systems. A simplest example of CSRF can be a the following: imagine that you’re logging into your favorite social network. When the authentication is done, the server will most likely establish a user session associated with the currently open browser window. Now, let’s say, someone sends you a link which looks the following: “http://mysocialnetwork.com/like?post=123”. The thing is that the developer of the website violated the behavior of the HTTP GET request - it’s designed to be idempotent, meaning that the requests of this type must only retrieve the data but not alter it. When you click a link, the browser redirects you to the the given URL where the session has already been established. Before you realize what happened, you “like” someone’s picture or product without explicitly doing so. A more sophisticated example may involve the use of session cookies (e.g. when you check “Remember me” while logging into the website) as well as the malicious website which contains a form, which, upon submitting it, sends a hidden request to the website where you authenticated yourself earlier. In this case the browser will happily pass session cookies along with the request and it doesn’t really matter whether this is a GET request or not.

One way you can protect your API from CSRF attacks is to use anti-forgery tokens, a functionality which is available in ASP.NET MVC and which can be easily leveraged in Web API. The idea is to have two tokens, which are going to be sent with every API request. One token is sent with the cookies, the other - with form data. These tokens later are going to be validated and if the validation fails, the client would get HTTP 403 “Forbidden”.

Let’s modify our previous example to support CSRF protection. First, let’s create a test view:

<script src="https://gist.github.com/volpav/ab7490f1405760293aa1.js"></script>

In order to protect our “Post” method inside the “PostsController” we need to have a way to validate the request before the action gets executed. With ASP.NET MVC you can just decorate the method (or the entire class) with “ValidateAntiForgeryToken” attribute but, unfortunately, this attribute has no effect when using Web API so we need an alternative. The way we’re going to approach this shortcoming is we’re going to write a custom authorization filter (I called it “ApiValidateAntiForgery” to eliminate any confusion with MVC):

<script src="https://gist.github.com/volpav/7774d860a044eb034fc3.js"></script>

The newly created attribute can now be applied to the “Post” action method like the following:

<script src="https://gist.github.com/volpav/688c3960e458b1db775b.js"></script>

Now if we submit the form, we’d get an error. In order to get rid of it, we need to supply authorization tokes. This is done the same way as with ASP.NET MVC, by putting the call to “Html.AntiForgeryToken” method inside the form:

<script src="https://gist.github.com/volpav/25f78a4cd0690bc18da3.js"></script>

Now the request passes the validation and we get the response from our API.

In cases when the call to the API needs to be made using AJAX, we’d use a slightly different solution - we’d pass the tokens with the custom request header. Here’s how it’s done on the client:

<script src="https://gist.github.com/volpav/dadf83a183e211e28714.js"></script>

Also, we now need to modify our validation logic to take the “RequestVerificationToken” header into consideration:

<script src="https://gist.github.com/volpav/77a96332ecd18b7ed319.js"></script>

After performing these changes, our solution is going to work both with normal submission as well as with AJAX-based one.

## Request Throttling

Request throttling is a way to limit the number of web requests according to a given rate. When working with Web APIs the term that corresponds to request throttling is either “Rate limiting” (for example, Twitter or Facebook), “API limits” or “Quotas” (for example, Google), or “API throttling” (for example, Amazon). Request throttling helps making your API more scalable by refusing requests from clients that are currently over the limit and therefore, “making room” (keeping resources available) for those clients who’re within the given rate. An example of a rate limiting can be the following statement: “Any particular client is only allowed to make up to 10,000 API requests per month”. By “client” I don’t necessary mean the physical machine, it can also be specific customer account (referenced by an API key, username-password pair or something else that can be used to uniquely identify the client in the system).

Request throttling is not natively supported in ASP.NET Web API but it’s very easy to implement the basic functionality using a custom HTTP handler. If you’re familiar with ASP.NET MVC, you can think of HTTP handlers as something which is very similar to action filters. Handlers get executed quite early in the pipeline allowing the developer to easily abstract the infrastructure functionality (such as rate limit checking) from the business logic and make the application code more modular and less coupled.

Let’s take a look at the example. We will start with defining a type that will represent an API limit:

<script src="https://gist.github.com/volpav/cd19f4027ff7f4a1c9e7.js"></script>

Next, we create our custom HTTP handler by deriving from “System.Net.Http.DelegatingHandler” and overriding “SendAsync” method:

<script src="https://gist.github.com/volpav/784764ec051db4af17b8.js"></script>

The high-level logic is pretty straightforward: before each API method call, we’re retrieving API limit for a given client and checking whether the limit has been exceeded. If so, we’re aborting any further execution by returning HTTP 400 “Bad Request”. Otherwise, the execution continues normally.

There’s a bunch of helper methods that are used inside “SendAsync”. Here’s the implementation:

<script src="https://gist.github.com/volpav/003c84dbb6294242bffa.js"></script>

As you can see, first of all we’re using a static in-memory dictionary to maintain the list of API limits together with the current usages per client. Of course, this is very naive implementation since this data will be lost with every app pool recycle but it’s here just to demonstrate the concept. Another thing worth mentioning here is that we identify the client by his IP address (but, as I mentioned earlier, the way you do this depends on what you “target” by throttling). For this purpose we use the “GetClientByIpAddress” extension method:

<script src="https://gist.github.com/volpav/580c90dad9de47f05d51.js"></script>

We can now plug our custom handler in by adding it to a “MessageHandlers” collection which is available on “GlobalConfiguration.Configuration” object (this must be done on application start).

## Long-running Tasks And Progress

Long-running tasks always looked to me as kind of a Holy Grail for many web developers. Every now and then you see questions being asked on StackOverflow as well as new blog posts being written about how you can incorporate this kind of functionality into your ASP.NET application. Most of these techniques and solutions, though, are quite fragile and can’t really be used in a real world scenario. The reason is simple - HTTP is stateless and any good framework (such as ASP.NET MVC) should embody this stateless nature and, by doing so, it encourages you to write more scalable and robust web applications. In other words, ASP.NET wasn’t designed to support this kind of scenarios. For example, there’re multiple different reasons why app pool can recycle and you’d need a lot of extra functionality to make your long-running tasks sustain this behavior. Any work that requires a substantial amount of time (as well as additional amount of computational power) to complete must be off-loaded from the front end of your web application.

For service-oriented applications, one of the ways to be able to have long-running tasks can be having a dedicated service for this purpose (let’s call it a background worker). If you’re working with Microsoft stack, a really nice option (in my opinion) is to create a WCF service and host it outside of the HTTP context (e.g. by hosting it inside the Windows service) so all the possible issues mentioned earlier will be avoided. Another good alternative is to use .NET Service Bus, although I’m not going to cover it in this article.

Now, how do you make it possible for the consumer of your service to know what’s the progress of performing a given tasks (how far did it get, when is it going to be completed, etc.)? The simplest solution is to periodically query the service for the progress information. While this will work, this is also not very scalable solution since we don’t want to bombard our background worker with requests while there hasn’t been any progress so far. The more correct way to solve this problem is to allow the service to “push” information about the progress (when it’s available) back to the client. With WCF this can be achieved by using callback contracts together with one-way operations.

As an example, I’ve created a simple application which parses a structure of a given website by following all internal links. The application consists of WCF service, hosted inside the console app, as well as ASP.NET MVC front end with some Web API endpoints for some AJAX-based scenarios. While I’m not going to demonstrate the entire source code here (it’s available at [https://github.com/volpav/webapi-sitemapservice](https://github.com/volpav/webapi-sitemapservice)), I’d like to show you the key parts of the application.

First of all, let’s look at the service contract definition:

<script src="https://gist.github.com/volpav/b5404772f185efe21f82.js"></script>

As you can see, we have two contracts: one is implemented by the service, another one (callback contract) - by the client. The callback contract can be retrieved by the service using “OperationContext.Current.GetCallbackChannel” construct. This enables the service to call back the client the similar way the client would call the service.

Here’s the part of the consumer functionality that enables communication with the service:

<script src="https://gist.github.com/volpav/3a987c5b080c2b9f24f1.js"></script>

For the simplicity, I’m storing the progress information (as well as all the results) in a static in-memory dictionary: the two calls -“SitemapManager.Current.OnProgress” and “SitemapManager.Current.OnCompleted” - simply update the data in the dictionary whereas the Web API endpoints access this data (instead of accessing the service layer directly).

## Conclusion

No doubt, ASP.NET Web API is a great tool for those looking to create their own APIs for the modern Web. This framework addresses many different use-case scenarios and is quite flexible to accommodate to almost every need. I hope that, by reading this article, you learned something new and that some of the information presented here will be useful in your daily developer adventures.

Below I’ve gathered some links to resources that can help you get a better understanding of topics covered in this article (some of the resources apply to ASP.NET MVC but certainly can be treated as guidelines for Web API as well):

- [Asynchronous actions](http://www.asp.net/mvc/tutorials/mvc-4/using-asynchronous-methods-in-aspnet-mvc-4)
- [OData](http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api/creating-an-odata-endpoint)
- [Batching](http://aspnetwebstack.codeplex.com/wikipage?title=Web+API+Request+Batching)
- OData & Batching - client-side libraries: [One](http://datajs.codeplex.com/), [Two](https://github.com/volpav/batchjs)
- [CSRF prevention](http://www.asp.net/web-api/overview/security/preventing-cross-site-request-forgery-(csrf)-attacks)
- [One-way calls, callbacks and events in WCF](http://msdn.microsoft.com/en-us/magazine/cc163537.aspx)
- [Long-running tasks and progress (example)](https://github.com/volpav/webapi-sitemapservice)

I also had a chance to present the above to my fellow colleagues as part of our “Lunch & Learn” initiative. Here are the slides: [http://bit.ly/191xhNp](http://bit.ly/191xhNp).