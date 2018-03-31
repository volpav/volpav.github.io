---
layout: post
title: Building Cloud Platform At The Bank
redirect_from:
  - /blog/building-cloud-platform-at-the-bank
---

Distributed computing _is_ hard to get right, as long as your system consists of more than just a web server and a database. Now, I can go and leverage tools like AWS, Azure or GCP and this engineering challenge becomes much more approachable. But what if I'm in a closed ecosystem such as a bank (of course, a result of being highly regulated and having strong controls in place to manage customers’ money)? Well, it wouldn't be that bad if banks weren't known to be slow, dated and inefficient when it comes to their tech (although financial institutions like Scotiabank have begun to change that image in the last couple of years). Modernizing a bank? It's definitely an interesting journey with many learnings so let's take a peek.

## Problem Statement

When I came to [Scotiabank](http://www.scotiabank.ca) last year, I joined a brand new department. Our ultimate goal was to help make this organization a true technology company, modernizing what exists and paving the way for rapid innovation (while meeting the requirements of regulators and maintaining the confidence of our customers). The challenge that I was given was one of the most ambitious: to help standing-up a general-purpose cloud platform that would allow application teams to care less about infrastructure and focus on creating amazing experiences.

## Getting Started

Now, where do you even start? So many different things come to mind, for example:

- The system needs to be reliable, secure and performant;
- It needs to integrate nicely with existing tools and processes;
- There needs to be a team that will maintain it as well as support its users;
- We need to "sell" it to developers, get them excited, train them if needed;
- ...


Despite all the great thoughts and ideas that are circling inside your head, what you should start with is: **figure out what already exists and why it doesn't work**. You don't want to be building this platform just because the word "cloud" is so trendy these days. On the other hand, stories like "it takes us a week to do a deployment" or "we have no good way of debugging this code in production so we just restart our JVMs every night" or even "we can't use Node.js because the internal ecosystem for it pretty much doesn't exist" - these should be music to your ears, it's something concrete you can work with!

Let's now consider three classical aspects: people, process and technology.

## People

This department is striving to hire the best local talent. We wear no suits and we stay away from internal politics. Our mission is of a cosmic scale yet we try to stay laser-focused. We're looking for great thinkers and problem solvers who have diverse experience with today's technology. It's true, though, that it's somewhat difficult for us to attract talent: hearing the word "bank" sometimes makes the opportunity seem far less attractive than it really is.

In every interview that I conduct these days, I always mention that, in regards to what we do, as much as it is a technology challenge, it is also a people challenge. Take this company's average software developer: you're most-likely looking at a person who has spent quite a few years at this bank, who knows his or her abilities and limitations (as a technical expert), who has an established way of doing things. When it comes to modern technology and cloud in particular, these individuals may have very limited to no experience with it. So how do you bring them along to this new reality? How do you educate them, and how do you turn them into your biggest advocates? You don't want to just tell them exactly what to do - rather, you want to give them the right tools and help them realize the full potential. Then, observe the magic as it happens.

## Process

When we started this, I don't think anyone on the team truly realized what we were getting ourselves into when it comes to working with existing processes. Yes, some of them do make a lot of sense (for example, application's cryptographic material is managed by a dedicated team). But how about the ones that clearly seem outdated, overlapping or redundant? And what about those that are just too slow, requiring a lot of manual effort?

The approach we've taken was: challenge everything and educate respective owners. With this in mind, it’s important to understand why things are the way they are and how it needs to work instead. Truth is: this is your reality so the sooner you learn how to navigate it (and later, make alterations to it) - the greater results you'll be able to achieve, quicker.

## Technology

This was (quite obviously) the most interesting part for me. Some of the biggest challenges we were encountering throughout the way were related to (surprise!) security. Now, just to make it clear: we've set up the platform on a public cloud and there's been a lot of work done to make sure all of it is highly secure and compliant with the right policies. 

But at the end of the day, to an organization that’s heavily regulated, it’s understandable that anything that is not inside a private data center has historically been considered non-secure by default. While management says they want to change this, established processes run deep. The implication of this? As mentioned earlier, we worked closely with our reviewers to help them understand: "there's nothing to be afraid of in this case". Yes, other times it really felt like trying to fit a square peg in a round hole and we had to compromise on a few things. But if you look at it holistically (and take into account that what you're looking at is not the final state of the system), we managed to get to quite robust implementation (and not just in terms of security).

From the automation aspect, what we’ve achieved is having the following in place:

- **Project-related**: requesting a developer account, provisioning a sandbox environment, managing your team’s access to the project;
- **Capacity-related**: auto-scaling policies, “disposable” environments;
- **Deployment-related**: ability to release new code daily, while meeting all of the existing control objectives (including code quality, security and audit);
- **Infrastructure-related**: on-demand services (e.g. cache), elastic runtime that requires no upfront provisioning or configuration, upgrade and backup pipelines;
- **Delivery-related**: thanks to all of the above, the process around obtaining production readiness sign-off for a new application is now much smoother since a lot of things are already covered;

## Where We Are Now

The active work on building the platform started in December, 2016 and by mid-May, 2017 (roughly just _6 months_ after) the platform was live. At the time of writing this post we have a number of applications running in **production** and serving **real customers**. We also have:

- Highly-available cloud platform with good support model in place;
- Rapid CI/CD pipeline allowing application teams to push their code into production on a daily basis;
- Comprehensive onboarding and training process and a team of engineers ready to help;
- Validation that what we imagined when we started this journey is possible and that this company is truly committed to transform itself technologically;

![Celebration](/assets/img/articles/celebration.jpg)

(on the picture above: celebrating first application's production launch)

## What Is Next

The work has already been started on the next iteration of the platform, making it even more reliable and secure, offering more services and expanding our user base. If you want to be a part of it (and, as I mentioned, there are other cool things that we're working on at this department) - [shoot me a message](/contact) and let's talk!
