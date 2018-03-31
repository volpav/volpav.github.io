---
layout: post
title: What To Expect From HTTP 2.0
redirect_from:
  - /blog/what-to-expect-from-http-2-0
---

It’s been over 14 years since the last version of the HTTP protocol (HTTP 1.1) was published in 1999. It would be fair to say that HTTP is an exceptionally successful protocol. It’s an application-layer protocol which means that it’s agnostic to things like transport (e.g. TCP, UDP) and session (SSL/TLS) which, in turn, makes it much easier to incorporate the use of the HTTP on variety of platforms. It’s also text-based which greatly helps us understand, debug and interpret its messages in our own applications. But the Web is evolving, and evolving very quickly. There are now many scenarios in which certain improvements to the HTTP would make our lives much easier and our applications much more secure, performant and scalable. That’s where the HTTP 2.0 comes to play. In this article I’m going to give you an overview of some of the features and capabilities that we should expect in the final version of the specification.

## Bandwidth And Latency

Let’s start by discussing the main idea behind HTTP 2.0. Some time ago several folks at Google decided to find out what are the bottlenecks in terms of the actual page load times. So, what they did is they took several large web sites, isolated them in an environment with the fixed latency (e.g. 100 milliseconds) and started measuring the page load times by gradually increasing the bandwidth while keeping the latency constant. The results were pretty interesting: when you go from, let’s say, one Megabit to two you see a 50% performance improvement. But as you increase the bandwidth further, those improvements get smaller and smaller. So, basically, after a certain threshold the bandwidth doesn’t matter anymore and what really becomes crucial in how fast your website is (in terms of resource load times and network utilization) is the latency that you have between the client and the server. Overcoming the latency-related issues is one of the main goals behind HTTP 2.0.

## HTTP 2.0 And SPDY

It’s fair to say that HTTP 2.0 is an evolution of SPDY, a protocol originally developed by Google. The main idea behind SPDY was very similar - to improve the overall performance of the web applications and to accommodate to the ways HTTP is leveraged these days. I think, SPDY had great success: Google themselves reported that many of their web applications (such as Gmail, Google Drive and Google+) got significant performance boost by leveraging SPDY. Other companies took notice and incorporated SPDY into their products as well (for example, Twitter, Facebook and WordPress officially support this protocol). Apache and Nginx web servers support SPDY via extension modules and the protocol is supported in all major browsers (including Internet Explorer). So, as I mentioned earlier, at least the first drafts of the HTTP 2.0 are largely based on the SPDY specification and these two protocols share a lot of concepts.

## TLS And Protocol Negotiation

I’ve seen some questions on the Internet regarding whether the web server must support HTTPS (via SSL/TLS) in order to be able to communicate with the clients over HTTP 2.0. To answer this question let’s examine the two possible scenarios:

- The client makes HTTP 1.1 request to the “http” URI.
- The client makes HTTP 1.1 request to the “https” URI.

Please note that here we’re assuming that the client (web browser) itself supports HTTP 2.0.

When the “http” URI is requested, the client will try to “upgrade” to HTTP 2.0 by sending the request headers similar to the following:

<script src="https://gist.github.com/volpav/18b0927d94f0d00ee1d0.js"></script>

In case the web server supports HTTP 2.0, it will answer with HTTP 101 “Switching Protocols” and all further communication will be performed via HTTP 2.0.

When the “https” URI is requested, the protocol negotiation will happen on the TLS level using the application layer protocol negotiation extension (TLSALPN).

There’s nothing in the specification that says that web servers must only allow SSL/TLS (and, therefore, refuse to serve requests over plain text) in order to be able to communicate via HTTP 2.0 (there’s a statement that says that the given HTTP 2.0 implementation must support TLS 1.2 but this doesn’t mean the client has to use it so it’s not the same thing). Although, it seems that there’s been a number of proposals among the IETF working group members to make this a mandatory. Furthermore, it seems that Firefox, for example, wouldn’t allow HTTP 2.0 over clear text (or at least, this was the case till recently - please correct me if I’m wrong).

So, to sum-up, it’s not clear yet whether your web application has to be HTTPS-enabled in order to leverage HTTP 2.0. For me, HTTPS doesn’t look anymore like a privilege but it’s rather such a normal thing to see on the Web these days so whenever you’re publishing something - make it a bit more secure, add SSL/TLS layer.

**Update**: [Ilya Grigorik](http://www.igvita.com/) also informed me that the current plan for Chrome is to support HTTP 2.0 via TLS only.

## Request Multiplexing

HTTP 2.0 introduces a feature called request multiplexing. What it allows is to maintain multiple HTTP requests concurrently. To better understand how HTTP 2.0 multiplexing works, let’s start by examining how HTTP 1.1 deals with multiple requests.

When the client issues an HTTP 1.1 request the web server has to fulfill it before any other request from the same client can be sent via the given TCP connection. Although, HTTP 1.1 introduced the default “Connection: Keep-Alive” semantics (the same TCP connection can be used for serving multiple requests/responses), it’s certainly not enough and doesn’t make your application nearly as scalable as it can be with HTTP 2.0.

Now let’s dig into how HTTP 2.0 maintains requests. First, there’s a concept of streams which is (citing the specification) a bi-directional sequence of HEADERS and DATA frames exchanged between the client and the server (don’t be confused by the HEADERS here as it doesn’t mean the “traditional” HTTP headers but rather the HTTP 2.0-specific ones). Also, we’ve just encountered one more important concept - *frames*. Frames are basically an atomic piece of data transmitted over the given stream. Every frame consists of a type, stream identifier, some flags, and, finally, the actual data.

Now the main beauty of all this is that with HTTP 2.0 multiple open streams can exist at the same time within the same TCP connection and so we can send data through these streams as it becomes available and not wait for an earlier open stream to close. HTTP 2.0 also introduces a flow control meaning that we can prioritize streams and therefore give a web server some hints on which data is more important for the client and which isn’t.

## Header Compression

It may sound a bit silly but HTTP headers can have a certain impact on the overall performance of your web server (or network infrastructure). They’re being sent with every request/response, their values change rarely so in most cases we’re simply losing the bandwidth by transmitting them. HTTP 2.0 introduces header field compression meaning that all HTTP headers are sent compressed and divided into so-called header block fragments. One thing to notice, though, is that HTTP header compression doesn’t preserve the relative order of header fields but personally, I can’t see this as an issue (if the application expects headers to arrive in a certain order, there’s certainly something wrong with the way it’s implemented).

## Server Push

Server push is another great enhancement that is coming with HTTP 2.0. This feature allows the web server to “push” resources to the client. There are several scenarios when this can be very helpful: for example, server push can be used when the web server “knows” the client will need a given resource (an image, a JavaScript file or a stylesheet) in the future. I’m not sure whether HTTP 2.0 server push has anything to do with the technique of the web server communicating back with the client when there’s a certain event happens on the server side but this scenario can easily be implemented using Server-Sent Events technology which is supported by all the major browsers (although, not by Internet Explorer - what a shame), standardized by W3C and plays nicely with SPDY (and so it should with HTTP 2.0).

## Conclusion

HTTP 2.0 looks very promising and will certainly not only improve our web applications’ performance but also improve the way we design and build them. We will no longer need to concatenate our JavaScript and CSS files in order to optimize network utilization, we also won’t need to use image sprites and implement domain sharding. I’m personally looking very much forward to when the specification will be finished and companies will start adapting to it.

At the end, some links to study:

- [Ruby podcast with Ilya Grigorik (Google) about HTTP 2.0](http://rubyrogues.com/135-rr-http-2-0-with-ilya-grigorik/)
- [HTTP 2.0 specification working draft](http://http2.github.io/http2-spec/)
- [Transport Level Security (TLS)](http://en.wikipedia.org/wiki/Transport_Layer_Security)
- [Difference between HTTP pipelining and multiplexing](http://stackoverflow.com/questions/10480122/difference-between-http-pipeling-and-http-multiplexing-with-spdy)

Also, just a day before this article was published, Mark Nottingham (from IETF HTTPbis Working Group) wrote an excellent blog post on the very same subject (even the title of the article is similar :-). [Check it out!](http://www.mnot.net/blog/2014/01/30/http2_expectations)

I also gave a small talk about HTTP 2.0 at work. You can download the slides from here: [goo.gl/fMqjf5](https://goo.gl/fMqjf5).