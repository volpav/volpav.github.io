---
layout: post
title: RESTful Server-Side Caching
redirect_from:
  - /blog/restful-server-side-caching
---

As per [Wikipedia article](http://en.wikipedia.org/wiki/Representational_state_transfer), one of the constraints defined by the REST methodology is that server responses can be cached by clients. This is typically achieved by sending out a bunch of HTTP headers that specify the cache policy and allow servers to validate the resource against its cached version. But sometimes you might want to cache parts of the resource while still returning the full response (and not responding with 304 “Not modified”). And by saying “parts of the resource” I don’t mean caching parts of the actual output but rather caching the data the server manipulates before sending out the response. 

## The Problem

A concrete example can be caching of database query results. Here’s a code snippet that gets a list of all news items from the given category in a hypothetical news reader application:

<script src="https://gist.github.com/volpav/4befd21ca1f5928a3dd5.js"></script>

So, as you can see this naive implementation uses ASP.NET session to store news items once they’re retrieved from the database so all the subsequent calls within the same session will hit the cache. While this is very simple (and not very pretty) implementation, it does the job of substantially reducing the CPU and I/O work once the items are stored in the session. What it lacks though is the ability to invalidate the cache (and give you fresh results). Of course, the heuristics will be dependent on the task (in this case I might want to keep news in the cache for 5 minutes and then load them again from the database) but what if I also want to let the client decide whether the cache must be invalidated? For example, I might hit CTRL+F5 in my browser got get the most recent news (and not wait for 5 minutes in order for the cache to be updated). While this is probably not something you’re going to use very often (for a number of reasons: not applicable to a particular task, the danger of having clients who either accidently or on purpose always request fresh versions of resources, etc.), this behavior seems to fit really well into the REST since by hitting CTRL+F5 (on an equivalent) you instruct the browser to discard its own cache and send some HTTP headers that tell the server (or a proxy) to invalidate the cache as well and give you a fresh response. Plus, you can of course send the appropriate HTTP headers when you’re performing an AJAX call to get the data (and by sending these headers you tell the backend to bypass the cache and get the most recent data).

## The Solution

What I ended up doing is I wrote a simple cache that honors “Cache-Control” and “Expires” headers and stores the data in the ASP.NET session (which makes perfect sense because you don’t want your global application cache to be cleared every time one of your users refreshes the page via CTRL+F5).

The cache provides the following methods:

- *Get* (returns the item with the given key).
- *Set* (adds or updates the item with the given key).
- *Remove* (removes the item with the given key).
- *Clear* (removes all items).
- *Contains* (determines whether the item with the given key is in the cache).

In order to respect HTTP headers while working with the cache, I implemented the following two methods:

<script src="https://gist.github.com/volpav/084dbdcf5597cceb318e.js"></script>

The above function is called every time the caller tries to add an item into the cache.

And here’s the function that ensures that the cache is still valid:

<script src="https://gist.github.com/volpav/8eb9b55db436b7c1e09d.js"></script>

The body of the function is only executed once per HTTP request (the function itself is called on every cache access). As you can see, here I’m also saving some CPU time by keeping the method result in a temporary storage.

One of the improvements that I incorporated initially was making it a LRU cache since I didn’t want it to grow in size infinitely. So there’s a default maximum capacity of 100 items and if the application exceeds this limit, the cache will start optimizing its size by removing the items that haven’t been accessed for a while. Also, each cache item can have an TTL (Time to Live) meaning that if the item has been in the cache for a relatively long time, it will be automatically removed during the next access operation.

Another improvement that I made is the ability to fall back to request-based cache in case the session-based version can’t be used. This is useful when, for example, you don’t want to make extra calls to the database within the current request (of course the items in the cache won’t surive the round trip but at least you have a small performance boost).

## The Code

I’ve put the source code on GitHub: [https://github.com/volpav/web-caching](https://github.com/volpav/web-caching). Feel free to fork it, play with it and submit pull requests.

## Known Issues

Here’s a list of known issues (and possible improvements) of my implementation:

- The LRU implementation can be optimized for performance.
- No cache pool to manage the amount of caches (a sort of LRU for caches in order not to occupy too much memory when there’re many concurrent users).