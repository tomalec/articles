Recently with Release Candidate of Starcounter, we deprecated a legacy feature from `<starcounter-include>` to support non-namespaced partial view-models.

That was a backward-compatibility layer for the times before we had the blending, before `Self.GET` was returning namespaces for all blended results.

However, it occurred this feature of `<starcounter-include>` was used over time to stamp non-blendable partials created in other ways - without `Self.GET` on the server-side.

I would like to show you why and how to transform those cases, to run happily after without problems and warnings.

## TL;DR
As warning in browser console shows, either:
 - Make it blendable by using `Self.GET` on server-side, or
 - Use [`<template is="imported-template">`](https://github.com/Juicy/imported-template) instead


## History

`<starcounter-include>` evolved with Starcounter's approach and set of server-side features over the years. At some point in the history, we didn't have the server-side blending - attaching multiple responses to the same URI. Back then `starcounter-include` was mostly the `imported-template` that was asking external API to fetch some arbitrary config for masonry-like element [`juicy-tile-list`](https://github.com/Juicy/juicy-tile-list). That seemed to be appealing and simple enough solution at the time. However, it had significant flaws, which we luckily improved:
 1. That was really naive to think that bin-packing/masonry is the ultimate design for every set of HTML elements mixed from multiple apps,
 2. The external API call was not really the Starcounter-way to handle JSON view-model - we were detached from all the cool performance and consistency features of [Palindrom](https://github.com/Palindrom/Palindrom) (PuppetJs),
 3. Scoping. When it comes to mixing multiple apps' we needed to separate the independent view-models. From just HTML perspective a list of elements from many apps concatenated together was not a problem, but to avoid conflicts in data-binding we need to have separate JSON (sub) trees.
 On server-side, we can hide the other apps' view-models merged/blended at some node/point to pretend that from an app author perspective it's just his/her tree from top to bottom. However, when it's serialized to plain JSON, an interim node that forks this view-model is needed. We need that not only to improve readability and debugging of such tree but simply to avoid conflicts, as two apps could use the same property name.


### Current behavior

Bin packing of elements (1.) was replaced with Shadow DOM - a Web Standard that not only lets app authors and designers use literally **any** HTML technique to style standalone or mixed apps, it also the standardized way to separate the layout related HTML composition from actual content and functional elements. For more read my previous articles: [Unobtrusive styling and composing 3rd party HTML content](http://starcounter.io/unobtrusive-styling-composing-3rd-party-html-content/) and [HTML partials/includes WebComponents-way](https://starcounter.io/html-partialsincludes-webcomponents-way/)

To address remaining problems we introduced simple namespaces for our partial view-models. Now, at every node where you attached blended results (blending point), you see:

```js
{
    YourApp_0: { /* your app's view-model */},
    AnotherApp_0: { /* view-model of another app that was attached/blendend to the same spot */ },
    SomeOtherApp_0: { /*...*/}
}
```

That gives us:
1. Independent scopes for data-binding,
2. Clear and explicit debugging,
3. Possibility to attach multiple responses from the same app to the same URI,
4. HTML Compositions with layouts could be provided by just an app, what improves modularity of entire solution,
5. We have a level (node) to be used by Starcounter's Platform, do store some data also on the client side, we use it, for example, to provide revision numbers for JSON Patch OT on the root level, so now no Palindrom metadata would collide with your app view-model.

### Backward compatibility - the source of current problems

To make it easier for app devs to adapt to the name-spaced version, we made this change backward compatible, so `<starcounter-include>` was still supporting partial view-models without namespaces.

## The change

At the beginning, we thought this will be really cosmetic change, a cleanup task to finally remove legacy code to support design patterns obsolete long ago. That was required not only for just code hygiene, but we needed to drop it to support actual features of `<starcounter-include>` for blendable, name-spaced case in a performant manner and with all border cases.

However, it occurred this backward compatibility bit was used as a feature by quite a few apps.

That even led to the problems of its own, like: "Hey, I saved a custom HTML Composition for this partial in my database and it's not being used!", "I cannot blend another app to this URI", which all led to the point that it was not possible, as it was not a "blending point". It didn't use `Self.GET` on server-side even if it used `<starcounter-include>` on client-side.

That's yet another reason I see for dropping support of non-namespaced partials - fail fast.


## The way to go

If you are currently using the `<starcounter-include>` for non-namespaced partial, you will need to update it really ASAP. But no worries, it shouldn't be that hard.

You need to ask yourself a simple question first.

"Do I want to let other apps blend in here?"

## "Yes"

If you include something that's conceptually separated from your main view-model, and this concept could be (mapped and) shared with other apps, I would advise you to make it blendable. That will let your app integrate tighter with others.

Then you need to update server-side code, to actually use blending point there. So instead of, for example
```c#
 post.Author = new BlogAuthor()
 // ...
```
you need to write
```c#
 post.Author = Self.GET<BlogAuthor>("/blog/authors/" + blog.author);
```
and add a handler for this new call
```c#
Handle.GET<string>("/blog/authors/{?}", (string id) =>
{
   return Db.Scope<BlogAuthor>(() =>
   {
       BlogAuthor page = new BlogAuthor()
       // ...

       return page;
   });
});
```

Your client-side code should remain as it is.

... unless you are using some CSS selectors or styling that assumes that `post.Author` will be only contain your app response. If it does you need to change that as other apps may potentially get stamped there as well.

For more on that please read [Blending guides](https://docs.starcounter.io/guides/blending)

## "No"

If your partial contains only your app internals, just modularized, and you are consciously don't want to use any Starcounter features like blending, custom Shadow DOM compositions fetched from the database, etc. If all you need is just a declarative client-side include, use [`imported-template`](https://github.com/Juicy/imported-template) - a custom element made exactly for it.

This time you keep your server-side code untouched. But, you need to stop using "blending point" in your client-side code.

Replace
```html
<starcounter-include view-model="{{subPage}}"></starcounter-include>
```
with
```html
<div><template model="{{subPage}}" href="{{subPage.Html}}"></template></div>
```

Naturally, you will need to update all your CSS selectors if you were matching this `starcounter-include`, to match `div` instead.

> Actually you may not need this `<div>`, but just to make it one-to-one change you can use this wrapper.

<!-- SELF NOTE: Add a section in docs to mention imported template, and how to create non-blendable partial -->


### Ceveat - declarative Shadow DOM

Unfortunately, technically, there is one thing that may not work - declarative Shadow DOM.

It's not a standard yet, and it is currently a `<starcounter-include>` feature.
We plan to make it an independent standalone element, but it's still in progress.


I would assume you didn't use it in your non-blendable partials, as you were not able to change it at run-time with the editor or with one provided from the database or blend with other apps.

If you did, you can:
 - move it into light DOM, no other app will get blended with you so the need of separating layout is not that big,
 - add it by yourself - just a `<script>yourElement.attachShadow({mode:'open'});//...`,
 - wait for W3C to [make it a standard](https://discourse.wicg.io/t/declarative-shadow-dom/1904),
 - poke me to finish [declarative-shadow-dom](https://github.com/tomalec/declarative-shadow-dom) independent custom element sooner.

## Do I need to change?

If you are not sure if you need to change anything in your codebase, we would recommend you to review all your `<starcounter-include>`s. However, to support you we are now throwing a warning to a browser's console in case we detect such a use.


## Summary

I believe, once you are aware of it, the change is not so problematic.

For sure it will let us make Starcounter's Web Platform more lightweight, running faster and be developed faster. I hope it will also make your code more explicit and easier to handle.


### Additional resources

 - `<starcounter-include>` release notes - [github.com/Starcounter/starcounter-include/releases](https://github.com/Starcounter/starcounter-include/releases)
 - `<starcounter-include>` README - [github.com/Starcounter/starcounter-include/](https://github.com/Starcounter/starcounter-include/
 - Custom element for client-side includes: [`<template is="imported-template">`](https://github.com/Juicy/imported-template)
 - Article on "HTML partials/includes WebComponents-way" [starcounter.io/html-partialsincludes-webcomponents-way/](http://starcounter.io/html-partialsincludes-webcomponents-way/)
 - Article on how Shadow DOM composition works "Unobtrusive styling and composing 3rd party HTML content" [starcounter.io/unobtrusive-styling-composing-3rd-party-html-content/](http://starcounter.io/unobtrusive-styling-composing-3rd-party-html-content/)
 - Blending guides [docs.starcounter.io/guides/blending](https://docs.starcounter.io/guides/blending)
