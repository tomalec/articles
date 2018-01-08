# I planned to write a year in review…


As 2017 ended, I planned to write a summary of my technical life. However, there is a single event that dominates my memories. The [TPAC 2017](https://www.w3.org/2017/11/TPAC/) - W3C Technical Plenary / Advisory Committee Meetings Week - was the best and most informative technical event I have ever attended. Not because it was the most recent one, but because of the number of deep details, new proposals, ideas, and really high goals to make the Web a better place in so many perspectives: for the authors, for users, from a technical and ethical point of view.
I feel extremely grateful and uplifted by having an opportunity to participate in such a big thing.

I'd like to share with you what I found most interesting there and my experience as a participant or observer in just a few out of many meetings that took place that week.
That was my second TPAC, but I still consider myself as a newbie in there.
<!--more-->

## The trip

<img src="http://www.webexecutiveforum.com/assets/images/tpac-1400x1750.png" width="300" alt="TPAC2017 Logo" style="margin: 0 calc(50% - 150px);"/>

This year TPAC was held in the USA, so that was quite a trip for me to get there from Poland. Lesson learned - jet lag and the time difference is a problem for brain-intense trips. Next time, I will come earlier to adapt, before meetings start.
However, in general, traveling the world for better Web is definitely a nice thing to do. Sightseeing, meeting new people and cultures, and doing nerdy stuff mixes well.

## Day 0

I almost made an off-by-one error and came too late for the pre-party for newcomers. If you think that W3C is a scary, closed, monolith, consortium made of big corp. politics - as I did a few years ago, I must tell you it is definitely NOT. It's an organization full of open, warm and caring people. They even make a party for a standalone newbie like me, so I could feel not so lost during the week.

## Day 1 - CSS - Grid ❤

Now it's time to notice that those nice people are also one of the best professionals in any web-related tech I ever met. That applies to every single day and Working Group.


On Monday I attended CSSWG meeting as an observer. Among all other topics, there were a few related to my beloved [CSS Grid Layout](https://www.w3.org/TR/css-grid-1/). Thanks to that many issues were resolved, such as intrinsic sizing of `overflow`ed grid and flex items.

Grid gaps for intrinsic(auto) sized grid containers, that are expressed in `%` are now resolved to `0`. This gives us consistent and more comprehendible speced behavior. It's fairly easy to remember that `20% * unknown` resolves to `0`, but predicting the `20% * grid container that is stretched by dynamic contents of its auto-sized items` is an extreme puzzle even to the author of such CSS.

One may say that migrating float-based layout, where you have columns and gaps sized in %, for an auto-sized container is now impossible. But if you really need to support a case of the unknown with, you can still create "gutter" tracks instead of using gap. Like: `grid-template-columns: 30% 5% 30% 5% 30%;`

Thanks to that percentage sizing of columns and rows could work exactly the same, just usually height of grid container is intrinsic (gaps will collapse to `0` unless you specify the `height`)


### Dev meetup

At the evening W3C organized a meetup for the local community. It was full of demos and followed by great talks. I suggest you check [www.w3.org/2017/11/Meetup/](https://www.w3.org/2017/11/Meetup/)   there are slides for most of them.

The one that interested me most is "HTML re-imagined for the era of Web apps" by [@leaverou](http://lea.verou.me/) - there is a video of similar talk at [www.webdirections.org/blog/video-week-mavo-html-re-imagined-era-web-apps-lea-verou/](https://www.webdirections.org/blog/video-week-mavo-html-re-imagined-era-web-apps-lea-verou/)

She introduced [Mavo](https://mavo.io/). I'm not picking a hype on yet-another-JS-framework. But I'm hyped that there are more and more tools and frameworks embracing a declarative approach to write Web Apps. The approach we love and advertise in [Starcounter](https://starcounter.com/) too. I think it should play even better with our [Palindrom](https://github.com/Palindrom/Palindrom) approach of data sync, and declarative, real-time database access.

## Day 2 - CSS


### Spatial navigation

We all know, that Web evolved from reading each other's text documents online. Back then single dimensional navigation form moving forward and backward (with tab or arrows) was perfectly enough, later the mouse/pointers helped to navigate in 2d space full of different UI elements. But sometimes you don't want (making breakfast) or simply cannot (a11y) use pointer, and moving forward and backward in modern Web is not enough, not to mention that in many cases even that is just simply broken. Hey, it's 2017! we have WebVR, HTML Elements rendering 3D objects, and we still cannot cover navigating in 2d?

LGE presented a nice proposal. It was initiated by TV needs, but the idea is now continued at WICG [github.com/WICG/spatial-navigation](https://github.com/WICG/spatial-navigation) to design solution to fit across all disciplines (CSS, Web Components, A11Y), to improve navigation around Web in general. Hopefully, it would let us improve the experience of re-arranged UI elements from many apps on the same screen.

### Archeology - scrollbars

Did you know, that there is a 17years old issue [bugzilla.mozilla.org/show_bug.cgi?id=77790](https://bugzilla.mozilla.org/show_bug.cgi?id=77790), related to the feature of IE 5.5 introduced in 2000? Something that was a pain for all of those years - styling scrollbars - now will get improved [drafts.csswg.org/css-scrollbars-1/](https://drafts.csswg.org/css-scrollbars-1/) Probably we will start with a small set of properties to set, to have interoperability between browsers, some of which ;) could like to preserve consistent UI for their users. Anyway, I find it a big step forward.

### List markers

How many times did you try to style list item markers? How many hacks do you know, to hide default bullet? How many times you saw/used `<div>` instead of a `<li>`? Thanks to the efforts of CSSWG there will be a new spec to solve that: [drafts.csswg.org/css-lists-3/](https://drafts.csswg.org/css-lists-3/)

### Power to the people! (Specificity)
...or actually web developers. That's one of two CSS bits I'm most excited about.

For years devs writing CSS selectors were bound and punished by the mysterious thing called specificity. I bet there are some devs even not aware that it exists, many who don't quite get how it precisely works - it took me few weeks to even learn how to spell it;). Don't get me wrong. I'm not trying to say specificity is bad. I think it's good as a default, for the beginning. For small sites, it works perfectly. The fact that many devs can create pages without even thinking about it, means that it somewhat does good.

However, for big projects, developed by many people separated into different teams or even organizations, or build out of 3rd party modules or frameworks, it becomes a hell. That's why people started to work around it by hacks in JS & CSS or with methodologies like BEM.

Lea Verou made a proposal [github.com/w3c/csswg-drafts/issues/1170](https://github.com/w3c/csswg-drafts/issues/1170) of a pseudo-class, like `:matches` but with zero specificity. That finally gives authors a control over it. Thanks to that you will be able to use very precise selector - not to affect too many elements, but at the same time reduce its specificity - to make it easily overwritable. This new selector - working name `:is` - will let you make parts of your selector match, but not increase specificity. For example, your general framework-ish selector that would like to apply some default look for enabled buttons: `.framework-scope button:not([disabled]):not([type=submit])` is way more specific than simple but precise selector to style your buttons `button.mine`. With the new proposal, framework authors could write selectors that are specific/precise but less intrusive (with smaller specificity), like: `:is(.framework-scope) button:is(:not([disabled]):not([type=submit]))`, which will have same specificity as just `button`



### Constructable Style Sheets

That's the other part of Tuesday's discussions I'm most excited about. Probably, because it's closed to Web Component's needs. How many times have you constructed style sheet in JS? If you are making Custom Elements, or just using Shadow DOM extensively, I bet you do that quite often. Thanks to new spec [tabatkins.github.io/specs/construct-stylesheets/](https://tabatkins.github.io/specs/construct-stylesheets/)] you could not only construct it in a more sane way with nicer API. You could do it without all that glue-code with creating/cloning `<style>` element and stamping it over and over again. Also, imagine how a browser could boost the performance of your site/app if it will be sure that you are passing the reference to the same object again. No need for stamping overhead (DOM events, mutations, etc.), parsing, processing, etc.

The spec is now in WICG, and there are browser vendors willing to implement it. I'm really looking forward to starting using it.

I hope it could improve the performance of Starcounter's blended views built out of dozens of partials from multiple apps, as we are repeating the same Shadow DOM styles for multiple instances of custom elements and blended partial views. I imagine it could also improve the tooling for the process of composing blended views itself.


## Day 3

That was the day crowded with [~40 shorter sessions](https://www.w3.org/wiki/TPAC/2017/SessionIdeas) on a variety of topics. I picked spatial navigation mentioned above and Web Platform Tests.

FYI there is [wpt.FYI](http://wpt.fyi/) - a dashboard with all the tests running in all major browsers. There are over a million tests running daily, monitoring interoperability of our working environment. So next time when you hit some weird behavior across the browsers, you can go to this dashboard and check whether one browser fails the test for given feature.

Even though the numbers look huge, there are still uncovered areas. Then, if your case is not covered by a test, go ahead and add one.
It will help browser implementers to fix it sooner and will help you to track the progress on this particular issue, so you could get rid of the workarounds. It's as easy as creating a PR for [github.com/w3c/web-platform-tests](https://github.com/w3c/web-platform-tests), and if you need more intro check [testthewebforward.org](http://testthewebforward.org/).

W3C folks are constantly working on improving the coverage, the ease of adding new tests, providing means to test more and more demanding features and APIs, that needs almost physical access to the device, and making effort to provide a tests suite with every change or a new spec.

I regret missing a [breakout on credibility](https://www.w3.org/wiki/TPAC/2017/SessionIdeas#Credibility:_Combating_.22Fake_News.22_on_the_Web). As you can see there is an ongoing effort to provide a means to deal with mis- and disinformation.

# Day 4 - HTML

Thursday was the day for the Web Platform WG to discuss HTML related topics.


The most interesting to me were asynchronous cookies API and Accessibility Object Model.

### Async cookies

How many times have you written/imported helper library just to read and write cookies? It's 2018, we have nice promise-based fetch API, IndexDB, Location API, but to read a single cookie we need to parse a huge string, not to mention problems with changing its value.

So there is a new proposal ([github.com/WICG/cookie-store/blob/gh-pages/explainer.md](https://github.com/WICG/cookie-store/blob/gh-pages/explainer.md), [patrickkettner.github.io/cookie-change-events/](https://patrickkettner.github.io/cookie-change-events/)) to provide API that will allow us to remove hundreds of redundant cookie access libraries, but also allow users to use cookies in service workers.

It's far from reaching the consensus on the full set of features, but at least two browsers expressed interest in solving the basic problems.

### AOM - devs ❤ a11y


Accessibility used to be a problem for many projects. Not only, because some people don't care and underestimate the need for it, but also the Platform was an obstacle.
Browers create accessibility tree separated from the DOM tree what introduces additional cognitive burden, that itself could be a reason for lazy developers to give up. But what is worse there is no way for the developers to effectively introspect this tree. All they have is just a limited set of mysterious attributes which can get only string values, usually ids which are extremely problematic...

Then what? Turn the voice over and smoke test?

For years browsers had a monopoly on the accessibility. Finally, developers could get an access to the Accessibility Object Model, and modify it as they do for Document Object Model.

If you are making Custom Elements, canvas based interfaces, using Shadow DOM or actually making anything more complicated that few elements with a small set of unique IDs you will be able to make it accessible without reimplementing the wheel and a full load of hacks. Also, you will be able to inspect whether you did it right.


[AOM](https://tink.uk/playing-with-the-accessibility-object-model-aom/) will be able to process references to objects, so you will no longer have a problem in big, modular apps with a uniqueness of ids, or elements hidden in Shadow DOM.

I raised the concern for the similar problem with regular `<label>` and _labelable_ (like `<input>`), that still requires ID string instead of an object reference. Hopefully, it will be addressed as well, in a consistent fashion.



## Day 5 - 🎉 Web Components ❤
<img src="https://raw.githubusercontent.com/webcomponents/webcomponents-icons/master/logo/logo_256x256.png" alt="Web Components logo" style="margin: 0 calc(50% - 150px)";/>

That was the day I was waiting for.

### Template Instantiation

It started with discussions on very fresh [proposal from Apple for template instantination](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Template-Instantiation.md). We all have seen tons of JS frameworks, they usually come with the templating system. Web Components already reduced the scope and need for JS frameworks by delivering a component model, DOM encapsulation and an inert `<template>` element.  But there was no simple, standardized way to bound data to a template and stamp it.

Hopefully, we will soon get the native way to do that, so you will not have to choose between 100's of frameworks that re-implements the same wheel, bother whether to use this or that syntax, which expressions can or cannot be used, etc.
You will no longer lock yourself with a given framework just because you need a simple way to provide some data to your `<template>`s.

In Starcounter we were in a need for a data-binding framework that plays nice with Web Components since 2013. Back then there was only one - Polymer. Right now, it's a little bit better, but there is still not much to choose from. Don't get me wrong, we like Polymer, as it's full of nice sugar for Web Components and it helps to create demos and prototypes really fast. However, as an application platform, we use it only for this particular reason - data binding. So, shipping an entire framework lead to many problems:
- There is a lot of unused code,
- Polymer is kind of moving away from MDV and `<template is="dom-bind">` way for building UI, towards writing a mediator element,
- Polymer is mostly focused on building the apps Polymer-way. So, fixing data-binding problems for our approach of mixing HTML `templates` provided by different vendors using different frameworks, is problematic and lasts for months.
- What is worse, it suggests that with Starcounter you are somewhat forced to use Polymer and you cannot build UI any way you want,
- That also blurs the lines between Polymer, Web Components and the way we sync the data between client and server - Palindrom.

That's why I really look forward to using this new template instantiation to remove hard dependency on Polymer from Starcounter's HTML app shell.

Also, it may help us simplify your implementation of declarative Shadow DOM.

There is also another good sign - that this came from Apple, so we are leaving the times where Chrome was the only browser to use Web Components features (w/o heavy polyfills)


### AOM

AOM was also discussed this day, pitching more Shadow DOM & Custom Elements specific problems.

### Shadow Parts

Since we have Shadow DOM, which encapsulates elements and styles, styling them became a problem, especially after `/deep/` and `::shadow` were removed. You may ask: "If you encapsulated them, why are you worried they are encapsulated and you cannot style them?" Yes, sometimes not putting into shadow is the answer, but in many cases, it's not. Remember the problem with styling scroll-bars? Their arrows, bars, and backgrounds were encapsulated and hidden for ages, so we complained like hell,  wrote custom ones with `div`s and JavaScript, just to be able to apply the styles. Same goes here.

Sometimes, you would like specifically expose some elements for styling. At the same time, you keep their markup in the shadow, so the consumer don't have to remember to put `<scroll-bar>` into your `<div>`, and you can still fully control your elements, attach listeners, add/remove them, etc.

For example, if you are creating `<spacecraft-control>`, you can expose an element from a shadow root: `...<div part="knob"></div>...`, so your element could be styled from the outside in every detail, without a need to rebuild the whole structure in the light DOM
```css
spacecraft-control {
    background: black;
}
spacecraft-control::part(knob){
    color: white;
}
```

Check out [the great blog post](https://meowni.ca/posts/part-theme-explainer/) made by Monica Dinculescu that describes it in more details.

This will finally allow us to create really stylable custom elements, then share them across many totally different apps.

The proposal also contains a great feature for theming: `::theme` which more-or-less lets you select all `::part`s regardless of its depth in shadow roots.


### HTML Import replacement

I cannot express how excited I was to see the discussions about HTML Modules - a replacement for HTML Imports.
Our approach to building system's UI out of composable, modular partial views, our client-side includes, with clean and structured `script`s execution flow is built totally on HTML Imports. That's why this is one of the most important topics for us.

You can take a look at Polymer's team [strawman proposal](https://github.com/w3c/webcomponents/issues/645#issuecomment-343601237).

The idea is to use ES Modules, as far as possible, not to re-invent the wheel. Then, to combine HTML Imports with ES Modules, to be able to use HTML Modules in ES Modules and vice-versa.
That means each HTML Module would expose HTML bits explicitly, what really aligns with the way we use and implement client-side includes.

There was long discussion full of new cases, ideas problems to solve. The WG decided to organize a dedicated meeting after TPAC in 2018 to continue the discussions.

I'm really looking forward ❤🍩.

I believe we will end up with not only a cool module mechanism to load "single file components", but an even better foundation for our composable modular views.

### Declarative Syntax (for Custom Elements)

Apple presented yet another proposal this time on [declarative syntax for Custom Element definition](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Declarative-Custom-Elements-Strawman.md). How cool is that? My two favorite features of Google's Polymer: declarative template binding and declarative way to define CE, are now being addressed by Apple on the spec level!

As there was a big number of comments, ideas, decisions and new problems to solve, WG decided to schedule this topic for the dedicated meeting as well. Especially given that was a Friday evening after the entire week full of intensive meetings.

I'd love to see Declarative CE syntax followed by Declarative Shadow DOM syntax, as we use and need it a lot. Personally, I think it's strange, that even though we finally have a way to encapsulate the HTML, we still cannot do that in... HTML. I suppose that confuses developers a lot, makes Shadow DOM not available for non-JS environments, makes it not available for the number of HTML developers who simply don't know JS, makes a burden many scenarios and decisions to consider, like when to attach it, how about a pre-attached state, etc.

It would be great to reduce the amount of boilerplate code needed to define an element. Usually less code => fewer bugs. It would also make the code more concise and readable. However, I'm a little bit worried that W3C efforts made for sugaring could slow down the pace of actually needed features that are not achievable today.

### What I find missing

I thought WebPlat would solve the status quo of customized built-ins. There are still in V1 spec, but there is no single browser that supports it. Apple strongly opposed and claimed that they will not implement it. Chrome shipped CEv1 without it, major polyfill made also by Google does not support it either, even though there is a PR that implements it, they don't want to merge and tangle customized built-ins too much due to the risk of future removal. Hopefully, since Mozilla recently claimed to ship CEv1 with customized built-ins, the status will change ca. March this year.

Also, speaking of Apple's Declarative syntax proposal, I expected there will be more about Shadow DOM, as I stated above. But maybe that's the reason of trying writing something on my own and changing what I've done in [`<template is="declarative-shadow-dom">`](https://github.com/tomalec/declarative-shadow-dom) and tangled inside [`<starcounter-inlcude>`](https://github.com/Starcounter/starcounter-include) to the actual prolyfill?


## Summary

That was really intensive and amazing week. I learned a ton, got inspired to do more things with Web, met many great people, build up even more respect and gratitude towards the goals and efforts W3C takes.


My advice/resolution to you in 2018 is:
- try a new native Web feature,
- if you don't know how something works in the Web - read the spec,
- if you know something is wrong with Web/could be better - contribute.

There is just a limited number of (luckily great, professional) people, trying to make it a better place for all of us.

Use it, appreciate it, share it and take care.
