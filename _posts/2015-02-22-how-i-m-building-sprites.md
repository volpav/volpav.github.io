---
layout: post
title: How I'm Building Sprites
redirect_from:
  - /blog/how-i-m-building-sprites
---

It’s been more than a year since I started working on [Sprites](https://spritesapp.com/), a tool for making infographics and online presentations. This post is about how I’m building the product as a single engineer/designer.

## At a Glance

I started working on Sprites back in September 2013. The early prototype has seen light in February 2014, in May I announced a public beta and the “version 1” of a product was released in September 2014. Right now we have over 10,000 users and we grow about 10% a month. We’re not profitable yet but plans are big and we believe that we’re moving in a right direction.

We are currently a team of two ([Denny](http://blog.spritesapp.com/2015/02/04/strong-teams-move-faster.html) is responsible for all the sales, marketing and PR). I’ve personally designed, implemented and continue to maintain 99% of the system, even though I’m looking forward to get some help with all this.

Quick note about the application: it’s a single web node which is hosted on Azure. We’ve recently become members of [Microsoft BizSpark](http://www.microsoft.com/bizspark/) and so I’m very happy that we can leverage Azure without too much worrying about our monthly bills (being able to use tools like Visual Studio for free is such a great perk as well).

## Technology

Since the majority of my professional expertise resides around Microsoft technology stack (particularly, .NET), it was an obvious choice for me to use this company’s tools for building Sprites. Now, I do like to explore other universes like Node.js, Scala, Rails and even a little bit of PHP but this time I had a different plan: to make something concrete, useful and functional without spending too much time learning (and fighting) the new framework, library or an IDE.

### Frameworks and Tools

Sprites is an [ASP.NET MVC4](http://www.asp.net/mvc) web application with the back-end written in C#. It’s a fairly standard set-up if you think in terms of middleware and an asset pipeline: I use default minification and bundling mechanism, the UI is just a bunch of partial Razor views and I don’t even bother loading them on-demand (it’s all still quite lightweight so I’m not worried). For interactions with the client-side code there’s a number of [WebAPI](http://www.asp.net/web-api) endpoints (more on that later).

On the front-end I use [LESS](http://lesscss.org/) for managing all the styles as well as [TypeScript](http://www.typescriptlang.org/) for all the client-side application logic. Now, it’s worth mentioning that I’m super-happy with how TypeScript makes it possible for me to write more robust and efficient JavaScript, faster. On top of all the great features it has (like modules, interfaces and inheritance, generics, etc.), there’s also an amazing support for it in Visual Studio (great IntelliSense, “Go to definition”, refactoring, auto-generation of source maps). I’ve been using TypeScript on a number of projects, large and small, and I never regretted the choice.

As for the JavaScript frameworks being used, the editor is built with [Knockout](http://knockoutjs.com/). Not because it’s is my favorite MVVM implementation but again, mainly because I knew it pretty well. I also wanted to start with a bare minimum, not introduce unnesesary complexity and not import stuff that I won’t need (speaking of Angular or Ember - definitely good candidates but felt to heavy when I was starting). I also use jQuery here and there, [Moment](http://momentjs.com/) and a coule others. The rest if completely custom (based on a concept that every piece can be reused which I later hugely benefited from when implementing real-time collaboration).

### Database

When I was sketching the data model and the overall flow of an application, it felt like NoSQL would do the job quite well: the entities were somewhat complex and so it made sense to represent them as graphs or documents, I needed fast writes for the good editing experience, I knew my models will evolve fast and so I wanted to avoid RDBMS enforcing any kind of schema, I wanted to use LINQ but I didn’t want to use Entity Framework (largely because it felt like shooting birds from the cannon).

I made some research and decided to go for [RavenDB](http://ravendb.net/). I chose it because it’s written specifically for .NET, very easy to set-up and painless to use in a default configuration, fully transactional, with configurable consistency model. For the record, I did use MongoDB on my [previous startup](http://priceflurry.tumblr.com/post/40419470714/goodbye-priceflurry), but was left with mixed feelings (its incredible ability to loose data even when there’s almost no load on the system, was making me furious sometimes). I also wanted to try Azure [DocumentDB](http://azure.microsoft.com/en-us/services/documentdb/) but found the pricing a bit too expensive.

### API

As I mentioned, since the entire UI gets loaded when you open Sprites editor, there’s only data that flows between the client and the server as you work with the tool. For this purpose I’ve created a number of WebAPI endpoints (again, fairly standard configuration). In order to make an API a bit more secure, I first made sure I only communicate through the SSL (my personal opinion is: why wouldn’t you want to use SSL on your website when it’s so cheap these days and also considering how other people and organizations invade into people’s privacy). Next, it’s all behind Forms Authentication which is driven either by standard username/password combo or via OAth (I’ve used [DotNetOpenAuth](http://dotnetopenauth.net/)). I’ve also implemented a sort of gate keeper which would check every incoming API request by examining its type, HTTP headers, cookies as well as a special, short-lived token (re-issued every now and then). If the request is note sane, it’s discarded. Now I think this gate keeper was a bit of an overkill but back then I didn’t want people discovering the API to abuse it so I’ve tried to make it a bit more secure.

### Background Workers

Sprites uses a number of background workers for things like email dispatching, video publishing, export, etc. These workers are currently implemented as Windows Services (I’ve used WCF boilerplate). The communication between the application and the worker is organized by using queues to make it all a bit more scalable. I have my own tiny implementation and I use [Amazon SQS](http://aws.amazon.com/sqs/) for some of the stuff. It all works quite well.

## Design

I don’t have a degree in design and I never studied design but I truly enjoy designing things myself. I’m not ashamed of saying that I borrow a lot of inspiration from other people. I don’t blindly copy the work of others though, instead I’d learn how the particular piece is done, why it’s done this way and then maybe try to implement something similar. I love all the aspects of web design in particular: how the main blocks are positioned, the choice of colors and fonts, the flow of an application, the user experience. I respect (and try to study) good copy-writing and I admire clear communication, pithiness and ease of use.

I design everything myself. Mostly, because I can but also because it’s hard for me to express to other people how I want this particular element to look like and behave and what kind of emotions it should trigger. But don’t think that I’m a rare species, a software engineer who can also deliver a visual aspect of a product. It’s been a long way of trials and errors and my first projects looked really ugly comparing to how I though they were looking.

## Delivery

As a startup, you need to be able to iterate quickly, delivering new features, bug fixes and other updates at a rapid phase. You need to consistently tweak your market fit, you need to please your early adopters, you’re too unstable to practice long development cycles.

I release new versions of Sprites several times a day. For now I don’t use any sort of continuous delivery (I’d go for [Octopus Deploy](https://octopusdeploy.com/), though). Instead, I use [Web Deploy](http://www.iis.net/downloads/microsoft/web-deploy) and just release from within Visual Studio. It’s super-convenient, fast and very easy to set-up.

### Working Overnight

It’s worth mentioning that I also have a full-time job. With this, I can only work on Sprites during night time and sometimes on Saturdays, so I have to maintain a high level of self-discipline and motivation: I don’t play video-games, I rarely go out, I get up at 5:30 am and I exercise at the climbing gym three times a week. Generally, I try to follow my own productivity tips but here I’m just saying that it’s very possible to combine the sort of things I mentioned (startup, full-time job and a few other bullet-points), you just need to prioritize stuff a bit more carefully.

### Meetings and Tasks

Meetings should be short, concise and on-target. I try to do a weekly live meetings at a nice indie coffee shop and I use [Slack](https://slack.com/) and Google Apps for Business for any other occasion.

While Slack is a perfect work chat, I use [Trello](https://trello.com/) to track all the tasks. Those that are product-related and involve writing some code, eventually become GitHub issues (I’d mirror Trello tasks in GitHub because it’s a nice way for me to track code changes in relation to an actual task but at the same time it’s much better to have a central place for all the work-related activities planned, whether this involves development hours or not).

## Conclusion

I’ve touched upon several aspects of me working on Sprites and I hope my overview was clear and useful for some. If you have any questions, feel free to ask them here or shoot me an email at volpav@gmail.com.