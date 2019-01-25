---
layout: post
title: Core Properties of Your Software
---

As engineers, when we discuss how to build a new piece of software, it’s often useful to try to answer the following question: **What are the core intrinsic characteristics that our system needs to have?**

Why do we need to ask this question? Software architecture is a lot about choosing one approach over another or, in other words, it is a lot about making compromises. The overall quality of your software will heavily depend on the compromises you made early in the process.

Think of it this way: let’s say you’re going to climb a mountain and you’re packing your backpack. What are the items you’re going to take with you? Warm jacket - essential. A stack of books from your home library? This can significantly slow you down so better to leave them at home. A pair of hiking boots? Certainly! An umbrella? There’s likely a more convenient way to deal with rain in this case.

You can’t have a software system that is extremely fast, secure, reliable, flexible, portable and simple _all at the same time_. You have to lower the bar around some aspects while meeting your core requirements.

For example, I would very much like my bank to ensure my digital accounts are properly secured whereas I can tolerate its mobile app being a bit slow sometimes. On the other hand, I wouldn’t expect their promotional webpage to require me to use multi-factor authentication in order to show me mortgage rates, it just needs to present information in a clear and accessible way.

When deciding on intrinsic characteristics of a software system it’s sometimes helpful to know “what’s in store” - have in front of you a list of those that you think about often. You can even go through the list and ask questions like “Do we care about this?”. If the answer is “Yes”, then you could dive deeper and form concrete requirements. If the answer is “No”, it’s helpful to elaborate on why you don’t necessarily need your software to have that kind of property. And actually, regardless of the answer, I highly recommend capturing the “why” - it’s going to save you a lot of time down the line (especially when your system is mature and went through a few large iterations, people might start questioning it’s core properties and have the initial rationale written down and readily available is going to help with re-assessments).

Here are some items that I try to keep in mind when going through design phase:

- **Correctness**<br />
This might surprise you but sometimes it’s enough for your software to be only approximately correct. For example, sometimes having highly-performant program which gives correct results in 80% of all cases is much-more desirable than having a slow program which is correct 90% of the time.<br /><br />
- **Security**<br />
  Do we store sensitive user data that we need to protect? Do we need to require authentication to access certain parts of the system? Are there different types of users that we need to treat differently? Do we need extra security given how we distribute our software (e.g. a mobile app which is on user’s device)?<br /><br />
- **Performance**<br />
Everyone likes it when software runs fast but sometimes you need to prioritize other things over raw performance. What is the nature of operations our program is performing? Should we optimize for reads or writes? Do we need to think about improving perceived performance (i.e. parts of the screen loading independently which allows the user to interact with an app before things fully load)?<br /><br />
- **Scalability**<br />
Related to performance, here you’d want to understand how you’re going to handle increasing system load (which might be based on the volume of requests, input/output data size, etc.). When we reason about approaches to scalability, often we consider horizontal (have more machines) and vertical (have more resources on each individual machine) scalability as some of the quick wins that can be taken.<br /><br />
- **Portability**<br />
Do we need to support multiple platforms (e.g. PlayStation and Xbox for games)? If the new platform is released tomorrow, should we be ready to support it (e.g. a new mobile operating system)?<br /><br />
- **Simplicity**<br />
Here it’s useful to think about the “cost of change” as a measure of how complex, time-consuming and risky it would be to introduce new changes to the system. When talking about user interfaces, we may want to choose one UI framework over another simply because it’s easier to use it (although, it might not be as powerful) and we have good knowledge around it. When looking at the back end, loose coupling and fine-grained components help with this quite a bit.<br /><br />
- **Extensibility**<br />
Here the main question is usually: should we allow altering functionality of our software with the logic that has not originated from our software (e.g. plugins and various extensions that take advantage of multiple extensibility points)? Depending on the degree of extensibility you may end-up with quite complex system so watch out for this one.<br /><br />
- **Accessibility**<br />
This is more applicable to user interfaces and behaviors. It's often a good idea to ensure your software is accessible to various categories of users (e.g. people with disabilities). Although, sometimes you might be designing something that is targeting a very specific persona (e.g. building a sonar with haptic feedback to help visually-impaired people navigate their surroundings).<br /><br />
- **Safety**<br />
In some cases you need to perform extra due diligence to ensure your software is safe for people to use. For example, when designing a program that controls an industrial press machine, you probably want to ensure an operator can’t accidentally activate it (which, in turn, may cause damage to other hardware or an injury to a coworker).

Hope this helps. If you have any questions, feel free to ping me on Twitter ([@volpav](https://twitter.com/volpav)).

Happy building!
