---
layout: post
title: Web Accessibility By Example
redirect_from:
  - /blog/web-accessibility-by-example
---

According to Wolfram Alpha, there’re currently around 2.2 billion Internet users which is roughly 40% of the world population. Around 10% of those 2.2 billion live with disabilities (blind people or people with low vision, motor-impaired people, etc.). With these figures in mind, it’s quite important to design your websites and web applications in such a way so they can be accessed by a broader audience. Today we’re going to talk about web accessibility and what you should do in order to make your websites more accessible.

## Reasons For Making Accessible Web Applications

Let’s start by asking ourselves: “Why do we need to make our websites more accessible?”. Well, first of all, it’s just the right thing to do. Internet, when it emerged, gave many people with disabilities a great opportunity to access and consume information they couldn’t consume before (for example, blind people couldn’t read newspapers). Another reason to build websites with accessibility in mind is that you’re actually making your website better (for example, by incorporating semantic markup). Also, it’s very important to follow accessibility guidelines when you’re working on a governmental website or a website for public organization or event since many countries (including U.S.A., Canada, Australia and New Zealand) have certain regulations that require websites falling into this category to be accessible to people with disabilities. Note that when your website is considered accessible, this usually also means that its content is better understood by machines which can make it stand out even more (for example, you’ll achieve better indexing by search engines).

## Identifying The Problem

Bring up your favorite browser (it doesn’t really matter whether it has built-in or plugged accessibility tools or not) and navigate to one of the web sites you developed recently. First, try using the website without the mouse (use only the keyboard). Notice how the focus changes when you “traverse” the page with the “Tab” key. Is the order of the focus logical? Does it “catches” hidden elements (links, buttons that are not initially visible)?

Try using arrow keys to select items in drop-down lists or pushing buttons/links with the “Enter” or “Space” key. Does the UI recognizes your input and responds to it the same way as it would respond to the mouse events?

If you’re using Google Chrome, try installing the ChromeVox screen reader extension and examining your website with it. Does the information it extracts from page elements make sense to you? Is it intuitive enough to use the website this way?

The above quick test will help you identify possible accessibility issues. Below are the most important goals we’ll try to achieve as per our accessibility walkthrough:

- Having a better focus management.
- Leveraging keyboard commands.
- Making custom controls (modal dialogs, sliders, etc.) more accessible.

Here’s how we’re going to approach those goals:

- By using clean, valid & semantic markup.
- By annotating elements with ARIA attributes.
- By properly handling keyboard events.

Let’s get started!

## Improving The Markup

Let’s take a look at the following piece of markup and try to see what’s wrong here:

<script src="https://gist.github.com/volpav/1b8ef178c043d95a1593.js"></script>

The problem here is that this markup is semantically incorrect: it defines a generic element whereas it’s meant to be a button. It may look and behave like a button (with little help of CSS and JavaScript) but the thing is that there’s no way for the screen reader (or search engine or any other program) to classify it as a button and therefore, to apply button-specific behavior to it. If you try using this element with only your keyboard (putting your mouse aside), you’ll also notice several issues: there’s no way to put a focus on the element (generic elements are not focusable by default) and there’s no way to “click” it (e.g. by pressing “Space” or “Enter” keys) unless there’s a JavaScript code that handles keyboard events.

The right way of defining a button would be to use either &lt;button&gt; or &lt;input&gt; HTML elements. If, for some reason, this is not an option (although it would be very strange it it isn’t), then we can enhance the element by first assigning it to an appropriate role as well as making it focusable:

<script src="https://gist.github.com/volpav/79411e7a6e29ff222808.js"></script>

Couple of things to mention here. First of all, we’re using the “role” attribute in the first snippet to indicate that the element represents a button. The attribute can contain multiple values separated by spaces (in case the element is in multiple roles). Some of the possible values are “button”, “link”, “textbox”, “banner”, “separator”, “slider”, “navigation”. The more complete list can be found here: [Semantics, structure, and APIs of HTML documents](http://www.w3.org/html/wg/drafts/html/master/dom.html#wai-aria).

We’ve also used “tabindex” attribute to make our element participate in page focus circles. Specifying “0” will make the browser decide (based on the order and the “tabindex” of other focusable elements) when the element gets focus. Specifying “-1” excludes the element from the list of focusable elements. The value which is a positive number means that the focus order is explicit (useful when the order of the elements on the page is not logical).

Here’s a couple of other examples of how a poorly implemented markup can be fixed:

Lists & navigations:

<script src="https://gist.github.com/volpav/63c94061a56efd4fd3b5.js"></script>

Sections & articles:

<script src="https://gist.github.com/volpav/0819de540d38724f7ac0.js"></script>

Tables:

<script src="https://gist.github.com/volpav/7c9a6cae502246bfec53.js"></script>

Labels:

<script src="https://gist.github.com/volpav/809bf1cc6c3e0878e6cd.js"></script>

## More On ARIA Annotations

ARIA states for *Accessible Rich Internet Applications* and the meaning of the term is to use certain methods and techniques in order to achieve better accessibility of the web content. As I mentioned earlier, the first thing you need to do when making your HTML element ARIA-enabled is to specify its role (using the “role” attribute) and, optionally, to specify its behavior using “aria-*” attributes. Note that many HTML elements have implicit ARIA semantics meaning that your browser automatically applies roles and behaviors to such elements based on their types (for example, the role “button” is automatically applied to &lt;button&gt; elements).

Here’s an example of accessible custom checkbox control:

<script src="https://gist.github.com/volpav/eeaba9a7300281bd1e4b.js"></script>

Of course, the same effect could be achieved with much less coding if we were to use standard HTML &lt;input type=“checkbox” /&gt; element.

ARIA attributes can be broke down into two main categories: states and properties (well, actually if you read the specification that it’s the other way around: states and properties are expressed via ARIA attributes). User agents handle states and properties differently and the main idea is that properties are not likely to change once the page is loaded (for example, the value of the “aria-label” attribute) whereas states imply the dynamic nature of the component (for example, “aria-checked” attribute).

The complete list of all available ARIA states and properties can be found here: [Supported States and Properties](http://www.w3.org/TR/wai-aria/states_and_properties). As with every specification, it might be a bit boring to read it (also, it really lacks usage examples) but I still suggest you to go through it just to familiarize yourself with what’s available.

## Handling Keyboard Events

We’ve already touched on keyboard events in the previous section where we were handling “Space” and “Enter” keys to dynamically update the “checked” state of our checkbox control. The proper handling of keyboard events is very important in case the user is not able to use the mouse. By “proper” I mean capturing (and processing) events only when it’s needed and makes sense. For example, if the widget on your page responds to “Key Up” and “Key Down” events, it’s important to not handle these events globally but rather when the widget is active (for example, is in currently focused). The other reason why events shouldn’t be handled globally (apart from it being a bad practice in most cases) is that some accessibility tools might also utilize keyboard shortcuts and so responding to an event that is not “meant” for your component/widget might lead to a lot of confusion from the user and a bad UX in general.

Let’s take a look at the custom select box which uses roles, ARIA attributes as well as “local” events to enable keyboard navigation:

<script src="https://gist.github.com/volpav/ecfc21b193595f5c364c.js"></script>

The main point here is of course the “onKeyDown” function which handles “Key Up” and “Key Down” keyboard events. Note that the processing only takes place if the select box is currently focused (the check for “gotFocus” field value at the beginning of the function body).

## AJAX and Dynamic Updates

Accessible web content doesn’t mean static web content and usually you don’t have to sacrifice functionality in order to achieve better accessibility (of course there can be certain trade-offs depending on how technically advanced your web application is). The WAI-ARIA 1.0 defines a set of live region attributes that can be used to mark certain elements of the page as live regions meaning that the screen reader (or any other accessibility tool) will constantly monitor the state of these elements and notify the user if the state changes.

First, let’s take a look at the following example which is not accessibility-friendly:

<script src="https://gist.github.com/volpav/acbd03d2fae93f9b132d.js"></script>

(Note: In the above code example I’m using jQuery to simplify access to DOM and XMLHttpRequest APIs).

What I’m doing is I’m sending an asynchronous HTTP POST request to add the given email to the subscription list (it will be an ordinary submit in case JavaScript is disabled on the client). While the server is processing the request, I’m displaying a “Please wait…” message to let the user know that his request is being processed.

The problem with above code is that it’s not accessibility-friendly. For example, there’s no way for a screen reader to figure out that I’m doing a background work so after clicking a button the user gets no (audio, in this case) feedback which is very confusing to say the least.

Solving the above issue is pretty simple: all we need to do is we need to add “aria-live” attribute to our progress indicator:

<script src="https://gist.github.com/volpav/17004356a02811d4ccc8"></script>

You can find the description of the “aria-live” along with other live region attributes here: [Live Region Attributes](http://www.w3.org/TR/wai-aria/states_and_properties#attrs_liveregions). There’re several levels of politeness that you can use with “aria-live”. These levels represent the order in which updates (comparing to other notifications) are coming to the user. In our example we’re using the “polite” level which means that the accessibility tool will wait until the current operation finishes before processing the live region update (for example, in case of screen reader, until it finishes saying the current sentence). You can use “assertive” politeness level if the update is important and the user must be notified right away (for example, when the user is being logged out of the system due to the long period of inactivity).

## Useful Resources

Here’s the bunch of links to resources where you can learn a lot more about web accessibility:

- [WCAG 1.0](http://www.w3.org/TR/WAI-WEBCONTENT/), [WCAG 2.0](http://www.w3.org/TR/WCAG/)
- [Accessible Rich Internet Applications (WAI-ARIA) 1.0](http://www.w3.org/TR/wai-aria/)
- [Web accessibility - Wikipedia](http://en.wikipedia.org/wiki/Web_accessibility)
- [section508.gov](http://section508.gov/)
- [Introduction to accessibility at Google I/O 2011](http://www.youtube.com/watch?v=lMrkCoqgoxw)
- [Advanced accessibility at Google I/O 2012](http://www.youtube.com/watch?v=iY-_tO_L3VQ)
- [Links to various online and offline accessibility checkers](http://www.w3.org/WAI/RC/tools/complete)
- [AccDC Technical Style Guide](http://whatsock.com/tsg/)

In case I missed something important here, feel free to mention those resources in the comments - I’ll gladly update this list.

## Conclusion

As you can see, it’s relatively easy to make your websites more accessible and the advantages of doing so are pretty obvious. The only thing that you need to keep in mind is that it’s much better to start with accessible architecture rather than trying to improve things when your website is nearly done.

I’ve also done a talk at my work about web accessibility. The slides from the talk are available here: [http://bit.ly/1cIlNm9](http://bit.ly/1cIlNm9).