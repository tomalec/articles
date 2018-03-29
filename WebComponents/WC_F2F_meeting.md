# W3C Web Components meeting

<!-- <br style="clear: both;"> -->
<!-- <img src="https://raw.githubusercontent.com/tomalec/articles/master/WebComponents/wcf2f/wlovec.jpg" width="300" alt="W❤️C sign on Takeshita street, Tokyo" style="margin:  0 calc(50% - 150px);"/> -->
<!-- <img src="https://raw.githubusercontent.com/tomalec/articles/master/WebComponents/wcf2f/wlovec.jpg" width="300" alt="W❤️C sign on Takeshita street, Tokyo" style="margin:  2em; float: right;"/> -->


This month [W3C Web Platform Working Group](https://www.w3.org/WebPlatform/WG/) gathered to discuss Web Components issues face-to-face.
The agenda was to resolve long-lasting contentious bugs, get consensus, gather browsers interest on new proposals and talk about the future of Web Components and Web Platform in general.


I must admit that the City of Tokyo, with it's Blade Runner-like streets enforces the feeling of building the future. I'd like to share with you what was the most interesting to me.

<!-- more -->

<img src="https://raw.githubusercontent.com/tomalec/articles/master/WebComponents/wcf2f/light_Pano.jpg" width="700" alt="Tokyo panorama in sunny day" style="margin: 0 calc(50% - 350px);"/>


## Contentious issues

We addressed a number of issues, some of them were few years old.


### [`ChildConnectedCallback`](https://github.com/w3c/webcomponents/issues/550)
Sometimes, you would like your element to react to children being added or removed. Currently to handle that custom element author have to use Mutation Observers, track `connected-`, `disconnectedCallback` and `slotchange` events carefully.

The group agreed that this is nice to have feature, but the performance cost of observing that for all elements could be expensive.
Currently, the problem is somewhat coverable by libraries and frameworks, therefore it has low priority. The missing piece of the puzzle seems to be imperative slotting API, which is more likely to be implemented first.

### [Base URL for shadow root](https://github.com/w3c/webcomponents/issues/581)
Hopefully, it would be solved together with HTML Modules.
Usually, the biggest problem arises when you provide a template with relative paths in an HTML module, then stamp it into an element inside the main document.


### [Prototype callbacks need no-ops](https://github.com/w3c/webcomponents/issues/582)
Given the number of classes, there is growing use of mixins and extending unknown classes. That's why you don't know if there is a `super` to run. The answer was to solve it in the future by [JS optional chain](https://github.com/tc39/proposal-optional-chaining). I'm really looking forward to reducing all those repetitive conditions.

### [Custom void/self-closing elements](https://github.com/w3c/webcomponents/issues/624#issuecomment-370310607)
We all heard the complaints about the verbosity of Custom Elements - the fact you need to explicitly close every custom element.

In short, nobody opposed the idea of self-closing elements, but the price of dealing with the parser is to hight for just a sugar of low verbosity. There are more urgent issues to be fixed around HTML and XML parser.

It will keep our code longer, but at least more explicit, shipped sooner and with stable inter-op.


### [Imperative custom elements upgrade](https://github.com/w3c/webcomponents/issues/710)

Wow, that was fast! Everybody agreed to have this, and here it is - instantly in the spec ([PR](https://github.com/whatwg/html/pull/3535))

Quite often when you are stamping the piece of DOM tree (in templating engine, from client-side include, etc.) you would like to make sure all elements are already upgraded in a detached tree. It could help reducing FOUC, simplify the data flow, etc.

Now, you can imperatively call `window.customElements.upgrade(treeRoot)` to upgrade all the elements.

In [Starcounter](https://starcounter.com/), it should let us have more control over partials being stamped, reduce FOUC and cascade of loaded partial views.


### [Custom attributes](https://github.com/matthewp/custom-attributes)

It was not discussed as a separate issue but was mentioned many times as a better alternative to some proposals. It looks promising for the group.

Maybe we as Starcounter should give it a try, and try to implement some features on top of that, instead of relying on for example customized build ins polyfill (independent `<declarative-shadow-dom>`)

## [Custom pseudo classes](https://github.com/w3c/webcomponents/issues/738)

That's yet another feature to make CEs stylable. [Custom psuedo-elements](https://drafts.csswg.org/css-shadow-parts/) will be discussed later and are already stabilizing. But this proposal is about exposing a **state of the host** itself. So you could implement your own states like, `active`, `expanded`, etc. Then, let your users use it from the outside for styling `your-fancy-element:state(expanded)`.

Browser implementers expressed significant interest, now we need just a formal proposal and polyfills :)

## [Scoped Custom Element registries](https://github.com/w3c/webcomponents/issues/716)

That's big!

I think everybody who tried to use Custom Elements in a bigger project faces the problem of custom element names collision, or at least version collision. So far all custom elements are registered in the same, one and only, global registry, given absolutely no way to scope.

Polymer made a proposal for low-level API to create separate scopes and registries. The proposal received interest from browser implementers. Even Apple expressed that foresighted performance limitations in the proposal could be optimized by the browsers. What is more, there are ideas to scope it on the tree DOM level, not only on shadow host.

That would be a great win for Starcounter's merged partial views. This would allow every app, even every individual partial view to use colliding custom element names with different implementations underneath. Possibly even in the light DOM!

## [Form submission](https://github.com/w3c/webcomponents/issues/187)

The standard to let custom elements participate in the form submission and actually writing custom form controls is getting pace. The proposal is available [as google doc](https://docs.google.com/document/d/1JO8puctCSpW-ZYGU8lF-h4FWRIDQNDVexzHoOQ2iQmY/edit?usp=sharing)
There are still many caveats to cover. Luckily browser implementers agreed to implement low-hanging fruit first. Something that would allow at least working around/polyfilling most of the use cases.

We should get `beforesubmit` event that would fire on the form element containing `FormData` so one could add/remove some imperatively. This would let us make submittable custom inputs, without a need to wrap native elements and wire them together manually.

## [Template instantiation](https://github.com/w3c/webcomponents/issues/747)

Day 2 started with templates. The discussions were focused on deep details and engaged almost the entire group. That, in my opinion, is a sign of maturity of this proposal. The biggest topic was how to express and expose the actual parts of the template.


See [slides from Polymer presentation]( https://docs.google.com/presentation/d/1f9lMbJA_TSUwXXWPvH7QcIEMy1yUVcSD_KNpGjHztyk/edit ) and [github.com/w3c/webcomponents#747](https://github.com/w3c/webcomponents/issues/747) for the summary.

What I like the most about the outcome of the discussions is that the Platform will start with shipping low-level primitives, that would allow frameworks and individual authors to address most of the use cases, then gradually deliver higher level features to provide nice and clean declarative and actually useful syntax at the end.

I believe with this low-level features in place we could implement our JSON Patch backed template binding. However, I would wait until we could use stabilized polyfill, and maybe one native (even flagged) implementation.

Even if we stick to the current solution with framework given template binding, we should eventually feel the benefits, as frameworks should adopt those low-level primitives soon. Then, use faster, native implementations. So far Angular expressed interest in adoption and Polymer is already experimenting heavily.

## [Custom pseudo elements, `::part` and `::theme`](https://drafts.csswg.org/css-shadow-parts/)

That's the "part" of stylable custom elements Starcounter needs the most.
I got the feeling that's really happening, and I am looking forward to seeing it implemented somewhere.

Discussions covered mostly how to sync with what CSS WG is doing, and the micro-syntax details for the `partmap` attribute (parts forwarding)
https://github.com/w3ctag/design-reviews/issues/230#issuecomment-371023989


## [Lightweight mechanism to add styles to a custom element](https://github.com/w3c/webcomponents/issues/468#issuecomment-370665411)

We also get a consensus on the way to attach default styles for a custom element. To do it without an overhead of attaching a shadow root, creating slot element and style element inside, just to create UA-like styles for a simple element that does not need Shadow DOM at all.

Technically it will not add anything to UA styles nor create a new cascading order, but observably would behave like one. I'd expect we could remove thousands of redundant elements from every page, and remove the overhead of multiple requests and parsing of CSS files.

```js
class MyElement extends HTMLElement {}
customElements.define('juicy-element', MyElement, { stylesheet: styleSheet});
// styleSheet could be referenced from <style>, <link> or new CSSStyleSheet
```
instead of
```js
class MyElement extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({mode: 'open'});
    this.shadowRoot.innerHTML = `<link rel="stylesheet" href="path/to.css"><slot></slot>`;
  }
}

customElements.define('my-element', MyElement);
```
Performance gain by making code cleaner and shorter - that's my favorite kind of fix.




## [Declarative Shadow DOM](https://github.com/whatwg/dom/issues/510)
That's the part I'm mostly sad about.

Maybe that's because I get emotionally attached to it due to the fact I aggregated a strawman proposal recently, implemented and supported it in Starcounter a few years ago.

It was nice to hear a lot of support from SkateJS and Salesforce.
Unfortunately for Google and Apple implementers support for HTML + CSS - no-JS devs and environments with blocked or nonavailable JS was not enough value and motivation.
Implementing `<shadowroot>` tag would require an additional specific step on end tag, what introduces lots of changes and is prone to security bugs, as it's tangled deep in the parser.

I could be sad but have to admit they have a reason to reject. That only gives more motivation, to explore and promote such solutions in user-land.
Hopefully, at least some of the use cases are solvable by library/custom-element. Syntactic sugar, HTML & CSS devs support, server-side rendering with just a little bit of JS. For the cases without JS we would have to find some solution that provides a view without Shadow DOM at all.

I think we should gather the community, use-cases and promote the benefits to improve the motivation for drastic changes in the implementations.


I hope we could get back to this topic once template instantiation will be more stable, as `<template>` element already solves most of the blocking issues mentioned by Apple. Initially, I just didn't want to overload the same element with many features (see `input`).

Maybe a second default processor fot tempalte instantiation - `<template type="shadowroot">` could be a solution.

Also, maybe, we could get back to it while discussing Declarative Custom Elements, as `<define>` has exact same problems with END_TAG callback as `<shadowroot>`. Browser implementers are more keen to ship Declarative CE. Maybe we could piggyback `<shadowroot>` to `<define>` in batch of HTML Parser changes ;)


Group non-enthusiastically agreed to put an explicit note that Shadow DOM feature does require JS to work.



## [HTML modules](https://github.com/w3c/webcomponents/issues/645)

The proposal is moving forward. More, and more detailed problems are being discussed, such as Base URL for a module, whether to use `Document` or `DocumentFragment`, parsing strategies, etc. Those may take a while to solve and get consensus on, but at least there still is implementers interest to cover this feature.

Just hope somebody will come up with a concrete proposal for all those cases.

## [Declarative Custom Elements](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Declarative-Custom-Elements-Strawman.md)

[Apple's proposal](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Declarative-Custom-Elements-Strawman.md) is really interesting and nice syntactic sugar to build Custom Elements. I think it could allow more web citizens to write their elements without getting into problems with JS bindings.

The idea actually makes sense only with template instantiation in place. So we would rather wait for templates before digging deeper into it.
Needles to mention its own problems with attribute vs. property binding, timing, exporting, etc.


Moreover, it also requires magical features for the end tag what was a blocker for the Declarative Shadow DOM. Given Declarative CE is just a sugar for something that usually needs scripting anyway, I don't think we should fight for it before other discussed features.


## Imperative slotting API

Even though there was no specific proposal or issue discussed during the meeting, the interest in imperative API for slotting was shared across all parties. It potentially solves many other issues and also plays nice with the approach: "Give web authors low-level primitives to do whatever they want".

Few days after Hayato Ito from Google posted his proposal at https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Imperative-Shadow-DOM-Distribution-API.md
It's definitely worth reading and providing feedback.



<img src="https://raw.githubusercontent.com/tomalec/articles/master/WebComponents/wcf2f/dark.jpg" width="700" alt="Tokyo panorama by night" style="margin: 0 calc(50% - 350px);"/>

## Summary

For me, that was most intense two days this year so far (Q1 2018). I felt my brain baked. However, I'd love to more such meetings. That's great to see Web Platform moving forward just in front of you. Talk to all those professionals. Meeting all those great individuals in person gives even more empathy, understanding, and respect, to their avatars seen on Github daily.


With love for Web, for Platform, [#foreveryone](http://www.foreveryone.net/).

<img src="https://raw.githubusercontent.com/tomalec/articles/master/WebComponents/wcf2f/wlovec.jpg" width="200" alt="W❤️C sign on Takeshita street, Tokyo" style="margin:  2em; ">
