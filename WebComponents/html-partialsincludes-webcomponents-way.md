Posted at Starcounter.io - http://starcounter.io/html-partialsincludes-webcomponents-way/

----

# HTML partials/includes WebComponents-way

## Abstract

Most of the bigger web project are designed to be modular. We all know that Custom Elements spec provides some modularity on individual HTML Element level, but is that all Web Components can give? Definitely not! Thanks to all specs combined, you can build apps that are modular on higher level. You can build your app out of many fully featured HTML Documents that seamlessly interoperate together. You can replace all your framework-specific, server-side templating engines, advanced portal-like systems with just Web Platform.


<!-- [[TOC to be put on a side:]] -->

  - [Abstract](#abstract)
  - [Introduction](#introduction)
  - [Partial](#partial)
  - [Solution](#solution)
  - [Step 0 - HTML as a string](#step-0-html-as-a-string)
  - [Step 1 - Template](#step-1-template)
  - [Step 2 - Custom element that stamps](#step-2-custom-element-that-stamps)
  - [Step 3 - Load external document](#step-3-load-external-document)
    - [XHR/AJAX](#xhrajax)
    - [HTML Imports](#html-imports)
  - [Benefits](#benefits)
    - [Declarative and clean insertion points](#declarative-and-clean-insertion-points)
    - [SPA achieved with single line](#spa-achieved-with-single-line)
    - [Pre-loading/lazy-loading content](#pre-loadinglazy-loading-content)
    - [Nesting](#nesting)
    - [Dependencies](#dependencies)
    - [Scripts](#scripts)
      - [Scripts executed per instance](#scripts-executed-per-instance)
      - [Scripts executed for all instances](#scripts-executed-for-all-instances)
    - [Use the DOM outside of importable template](#use-the-dom-outside-of-importable-template)
    - [Styles](#styles)
  - [Further steps](#further-steps)
    - [Two-way data binding](#two-way-data-binding)
    - [Theming](#theming)
    - [Note on V0 vs. V1](#note-on-v0-vs-v1)
  - [Summary](#summary)
    - [Additional resources](#additional-resources)


<!-- keywords: Web Components, partials, include, import, template, binding, SPA, Web Platform -->

## Introduction

Most of you have probably heard of Web Components, but just to get everyone on the same page, Web Components are (mostly) shipped in all browsers - and they work.

> "Yeah, yeah we all have seen the fancy tutorials on how to use Shadow DOM, HTML Imports, Templates and Custom Elements."

Yes, that's why I would assume that everybody already knows that WC are a cool native alternative to making framework-specific widgets.

But how can we use them combined?
How they help you simplify the application design?
How can they help you do things that were not possible, or overcomplicated, before and to do it with just few lines?
Let's take a look at the benefits of Web Components for bigger projects.

For the scope of this article, I'll pick a single, but quite useful concept: **partial**

## Partial

This is not a formal term, so let me briefly describe what I mean.

Consider that you (or your colleague, vendor, or anybody really...) created a good looking piece of HTML app/page. Not just a widget or tiny Custom Element, but fully featured HTML thing - with styles, scripts, data binding and tons of awesome features. Now - you're writing another piece of awesome HTML and you would like to pick the previous thing and put it exactly _here_. ...aaand maybe _there_ as well. You know, like an `include` keyword in other languages. Some may say "Make it a Custom Element!", but sometimes you can't (it's not your code), or becasue sometimes it simply makes no sense.

Precisely, by **partial** I mean - a fully-featured HTML Document(Fragment) that uses anything that HTML spec allows and is full of working features. A document that you would like to use in other documents, and make them inter-operate.

## Solution

> **tl;td** - full solution is described at [this section](#html-imports) and obviously "there is a Custom Element for that" [imported-template](https://github.com/Juicy/imported-template)

Naturally, I could point you to a ready-made library that contains _X_kilobytes of code and does magic at unknown cost.

However, I'd like to show you that Web Components-way is the "simple and easy way", so let's implement it together step by step.


## Step 0 - HTML as a string

All of us know how to use `.innerHTML` property, right? Perhaps, for some extremely simple use cases that's enough.
Perhaps, most of you are used to it - as that's the way we've done it for ages: Fetch the string from AJAX/comment element/script element, and append it to the DOM.
Perhaps, most of current templating engines work that way.

But - there's so much more! Instead of settling, read the following to see how much more you can get.

## Step 1 - Template

Starting from the basics, we've read the [first tutorials](http://www.html5rocks.com/en/tutorials/webcomponents/template/) so we know how to use new `HTMLTemplateElement` (`<template>`):

<a class="jsbin-embed" href="http://jsbin.com/wotanu/embed?html,output">Live sample at jsbin</a>
<!--
```html
<body>
  <template id="my-partial">
      <h3>My partial</h3>
      Quite inline and not so external, but at least easy to use
  </template>
</body>
<script>
    var partialTemplate = document.getElementById('my-partial');
    var partialInstance = document.importNode(partialTemplate.content, true);
    document.body.appendChild(partialInstance);
</script>
```
 -->

OK, first step achieved, we are stamping document fragment. But, it's not very impressive as it's still local and we haven't even thought about dependencies, scripts etc., but at least it works with just a few lines.

## Step 2 - Custom element that stamps

Let's continue through [basic tutorials](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/) to make it more declarative and easier to describe - let's make it a Custom Element.


<a class="jsbin-embed" href="http://jsbin.com/fihefo/edit?html,output">Live sample at jsbin</a>
<!--

```html
<body>
  <template id="my-partial">
      <h3>My partial...</h3>
  </template>
  <stamped-partial ref="my-partial"></stamped-partial>
  Let's stamp it many times
  <stamped-partial ref="my-partial"></stamped-partial>
</body>

 <script>
(function(){
    var StampedPartialPrototype = Object.create(HTMLElement.prototype);
    StampedPartialPrototype.attachedCallback = function(){
      this.stamp(this.getAttribute('ref'));
    }
    StampedPartialPrototype.attributeChangedCallback = function(name, oldVal, newVal){
        if(name === 'ref'){
            this.innerHTML = ''; // clear old content
            //stamp partial
            if(newVal){   
              this.stamp(newVal);
            }
        }
    }
    StampedPartialPrototype.stamp = function(id){
        var partialTemplate = document.getElementById(id);
        var partialInstance = document.importNode(partialTemplate.content, true);
        this.appendChild(partialInstance);      
    }
    document.registerElement('stamped-partial', {
        prototype: StampedPartialPrototype
    });
}())
</script>
```
-->

If you want to stamp it exactly where you put the element, not inside it, you can use `insertBefore` instead of `appendChild` - and need to take care of stamped nodes after your element is detached (w/ `detachedCallback`). See [http://jsbin.com/fefeya/edit?html,output](http://jsbin.com/fefeya/edit?html,output).

Ok, we have a much nicer API and declarative code, but we're still missing core features.

## Step 3 - Load external document

Usually you would like to pick your partial either from:
- external file,
- server-side (dynamic) generator,
- 3rd party.

### XHR/AJAX
Simple! We can use good old XHR (AJAX) to fetch it.

We will send a request fot ir on `attributeChangedCallback`, then stamp it as before ([Step 0]/[Step 1]).
> If that is enough for you you can pick already made [`juicy-html` element](https://github.com/Juicy/juicy-html) - it supports template tag, inline input, external files, native and Polymer's data-binding.

But, now we should start to think about more sophisticated use-cases and bigger projects. What about:
 - execution of loaded scripts (you know, `.innerHTML` method will not work for `<script>` elements),
 - nested partials,
 - dependencies,
 - duplicate requests,
 - caching?

Again, Web Components comes to the rescue, with ...

### HTML Imports

If we load an external document as `<link rel="import" href="my-external-partial.html">`, thanks to HTML Imports spec:
 - we could use `<script>`s, `<style>`s, -and everything,- (I don't understand this?) as it will be executed as regular HTML Document,
 - there's no problem with using our `<stamped-partial>` Custom Element inside the imported document (sub-partials),
 - nested dependencies and nested HTML Imports will be resolved by browser...
 - ... as well duplicates...
 - ... and caching.
 - we can also put templates there - but you will see this huge benefit later.

From what we've learned from [HTML Imports tutorial](http://www.html5rocks.com/en/tutorials/webcomponents/imports/) we can stamp the element from imported documents.

I would like to introduce single, small constrain or convention on what we will import: only `<template>` element.
This is mainly for code clarity and consistency, but you'll soon notice that it gives even more flexibility and opens a big box of cool features.

Here's the small change needed in our custom element definition:

**StampedPartial.html**
```js
//...
StampedPartialPrototype.stamp = function(href){
    // create HTML Import
    var link = document.createElement('link');
    link.rel = "import";
    link.href = href;
    var element = this;
    link.onload = function processImportedDocument() {
        var partialTemplate = this.import.querySelector('template');
        var partialInstance = document.importNode(partialTemplate.content, true);
        element.stampedNodes = Array.prototype.slice.call(partialInstance.childNodes);
        element.appendChild(partialInstance);
        // or to stamp it on the same level
        // element.parentNode.insertBefore(partialInstance, element);
    }
    document.head.appendChild(link);
}
//...
```
[jsbin with full code sample](http://jsbin.com/hasewab/edit?html,output)

That's it!

## Benefits

Now, ponder for a moment what it actually gave you.

### Declarative and clean insertion points

In your document you can stamp content, as simple as:
```html
  <p>Blah.. blahh..</p>
  <h3>My own div soup with</h3>
  <stamped-partial href="/path/to/externalPartial.html"></stamped-partial>
```
You can stamp it multiple times.
You can change the partial in run time, for example...

### SPA achieved with single line

```js
document.querySelector('stamped-partial').setAttribute('href', '/new/page.html');
```

### Pre-loading/lazy-loading content

By adding HTML Import for given partial, before requesting to stamp it, the browser will download the resource, which could be used later.

```html
<button>Give me partial content</button>
<stamped-partial></stamped-partial>
...
<script>
    var lazyLoadedPartialURL = 'partial.html';
    // lazy load it on page load
    document.addEventListener('load', function() {
        var link = document.createElement('link');
        link.rel = "import";
        link.href = lazyLoadedPartialURL;
        document.head.appendChild(link);
    });
    // later on after user action, you may need it
    document.querySelector('button').addEventListener('click', function(){
        document.querySelector('stamped-partial').setAttribute('href', lazyLoadedPartialURL);
    });
</script>
```
<!-- :construction:  jsbin link to be provided -->

### Nesting

You can simply nest partials - it's just HTML

**index.html**:
```html
  <p>Blah.. blahh..</p>
  <h1>My own div soup with</h1>
  <stamped-partial href="/path/to/partial.html"></stamped-partial>
```

**/path/to/partial.html**:
```html
<template>
    <h2>Partial content</h2>
    <stamped-partial href="/sub/partial.html"></stamped-partial>
</template>
```

### Dependencies

Partials may contain dependencies, which will be resolved according to HTML Imports spec
**partial.html**:
```html
<link rel="import" href="bower_components/juicy-ace-editor/juicy-ace-editor.html"/>
<template>
    <h2>Partial content</h2>
    <juicy-ace-editor></juicy-ace-editor>
</template>
```

### Scripts

That's one of my personal favorites.

You can get great control over all required scripts. Like:

#### Scripts executed per instance

If you need code to be executed after the individual partial is stamped, and do so for every instance.
For example if you are using a non-webcomponentized widget:

**index.html**
```html
  <h2>example 1</h2>
  <stamped-partial href="/hot-sample.html"></stamped-partial>
  <h2>example 2</h2>
  <stamped-partial href="/hot-sample.html"></stamped-partial>
```

**hot-sample.html**
```html
<template>
    <div></div>
    <script>
        console.log('setting up Handsontable in ', document.currentScript);
        new Handsontable(document.currentScript.previousElementSibling);
    </script>
</template>
```
<!-- :construction: jsbin link to be provided -->

The script will be executed as any times as many instances of `hot-sample.html` partial we will (re-)stamp.


#### Scripts executed for all instances

Also, you may need to execute a script once for all instances. For example, load such non-webcomponentized library.

**hot-sample.html**
```html
<script src="bower_components/Handsontable/Handsontable.js"></script>
<template>
    <div></div>
    <script>
        console.log('setting up Handsontable in ', document.currentScript);
        new Handsontable(document.currentScript.previousElementSibling);
    </script>
</template>
<script>
  console.log("I'm the inline script that executes only once!",
    "and I come from", document.currentScript.ownerDocument);
</script>
```
<!-- :construction: jsbin link to be provided -->

Please note, that `Handsontable.js` will be executed only once, as - thanks to native HTML Imports de-duping - `hot-sample.html` will be loaded just once.


### Use the DOM outside of importable template

You can create and use the DOM outside of the template to be imported. For example, to be able to run `partial.html` standalone, or in case a non-webcomponentized library need some specific HTML structure.

**partial.html**
```html
<script src="bower_components/Handsontable/Handsontable.js"></script>
<template id="my-partial">
    <div></div>
    <script>
        console.log('setting up Handsontable in ', document.currentScript);
        new Handsontable(document.currentScript.previousElementSibling);
    </script>
</template>
<h1>Non-imported content</h1>
This could, for example, be used to render this document directly in browser.
This could also render template above using "local template" technique:
<stamped-partial ref="my-partial"></stamped-partial>
```
<!-- :construction: jsbin link to be provided -->

### Styles

 I think you've figured out that you can use inline or external styles everywhere you need.

## Further steps


### Two-way data binding

Two-way data binding can be easily applied to all of the above. Once the template is stamped, we are within the same document.
Take a look at linked, already made Custom Elements, or wait for another article.

### Theming
If partial content comes from a 3rd party or multiple vendors you may need to apply common consistent look and feel for it.

Ha! that's where Shadow DOM comes into the game.
There's also a clean solution, but I'll leave it for the future articles.

### Note on V0 vs. V1
Everything above was written in terms of Custom Elements V0 - the current version of the spec. Soon V1 - the new version - will be shipped. Luckily, the principles are the same so it will work with V1 as well. I'll post the V1 version of the article, once polyfills for V1 APIs will be released, so I could attach fully functional demos.

## Summary

As you can see, Web Components give you the opportunity to easily and in a clean declarative manner compose the web pages and apps, not only on Elements level but also on full Document(Fragment) level. Finally - you're able to do this simple thing that wasn't possible for years - include one HTML into another (with close interoperability, not just with isolated `<iframe>`)

Also, by conforming simple convention - expose `<template>` to be stamped - you have the complete freedom to do whatever HTML allows, building big, composable, modular, interoperable apps. All of that without bounding your product and yourself to any particular framework, without learning any additional language, using just what the Web Platform provides.


If you have any comments, questions, interesting use-cases, please contact me via comments here, [twitter @tomalecpl](https://twitter.com/tomalecpl) or any applicable github repo.

### Additional resources

 - Custom Element that covers all these features: [`imported-template`](https://github.com/Juicy/imported-template)
 - Custom Element with simplest features: [`juicy-html`](https://github.com/Juicy/juicy-html)
 - In-memory application platform that supports such partials [Starcounter](http://starcounter.com/)


<script src="http://static.jsbin.com/js/embed.js?3.39.18"></script>
