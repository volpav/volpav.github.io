---
layout: post
title: Caching HTTP Responses With Angular
redirect_from:
  - /blog/caching-http-responses-with-angular
---

I came across a nice little challenge the other day: we noticed that our Angular application can sometimes be a bit too chatty with our backend so we’ve decided to eliminate some of the unnecessary (redundant) API calls by caching initial responses on the client. Changing application logic was not really an option at that time so I tried to see how this can be done with the help of Angular middleware of some sort. Surprisingly, Google yielded no suitable solution, so I came up with my own which I’d like to briefly describe in this post.

## The Idea

The idea is to have an HTTP interceptor which uses the return value of [URL.createObjectURL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL) method as a substitute for the actual request URL so that instead of making a trip to the server, the user agent would instead load in-memory data.

Let’s see how the above can be implemented.

## The Implementation

First, let’s define a skeleton for our HTTP interceptor:

<script src="https://gist.github.com/volpav/8235a4a674ac264ab7a0.js"></script>

As you can see, we’re relying on browser’s support for “URL”, “Blob” and “WeakMap” APIs (we are pretty ambitious/experimental with the latter one - this is mostly for code clarity, though, and so this dependency can easily be eliminated). If there’s no support for a particular API, there will simply be no caching involved and no errors will be thrown.

Next, let’s take a look at the high-level implementation of the interceptor hooks:

<script src="https://gist.github.com/volpav/f3273263c6efd685177c.js"></script>

The logic is the following: if **on HTTP response** we can find a corresponding *cache rule*, we’d store the response data internally so that **on HTTP request** if the data has already been stored for to the given rule, we’d point to that data instead.

Now, let’s take a look at our cache rules:

<script src="https://gist.github.com/volpav/b3b6f10aea536db91fcc.js"></script>

We are pretty naive in a sense that we try to match the URL completely when we are being asked for a cache rule that corresponds to a given HTTP request configuration. That is okay, though - often, this is exactly what you’re looking for (and it was enough in my case so I decided to not over-complicate the code with unnecessary functionality like matching by pattern). Also, our “logout” rule has an “onInsert” handler which we use to actually invalidate the entire cache when the user logs out of the system.

Finally, the part that deals with the actual caching looks like the following:

<script src="https://gist.github.com/volpav/b236563030ee4d65e423.js"></script>

The most interesting part is the “set” method which makes a use of [URL.createObjectURL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL) method for making our data available via in-memory URL (and what we store ourselves is just this URL).

Here’s how to register our interceptor:

<script src="https://gist.github.com/volpav/dd184fdef4e3c61c1973.js"></script>

That’s it! The full source code is available here: [Caching HTTP requests with Angular](https://gist.github.com/volpav/fa48d57d5ff1c287a488).

## Conclusion

By using an HTTP interceptor we made sure our caching layer is pluggable and can be taken out at any time with zero impact on the rest of the codebase. Also, I find the use of object URLs to be pretty elegant solution in this case (although, please, [do let me know](https://twitter.com/volpav) if there’s a better way of implementing request caching).