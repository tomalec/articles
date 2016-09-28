# Unobtrusive styling and composing 3rd party HTML content

## Abstract

In modern, modular web apps it may happen that you are given the set of HTML Elements to be stamped to your Document (Fragment) - for example from a [partial](http://starcounter.io/html-partialsincludes-webcomponents-way/). Usually, you would like to preserve consistent look and feel within you app, so you would like to mess with them a little - compose, apply styles, etc. The problem starts if simple CSS is not enough and you cannot re-arrange them. They may rely on order or position in DOM to interoperate, you would try to keep data-binding simple, or you just do not want to expose huge style-related div-soup into your markup. Luckily we now have [Shadow DOM](https://w3c.github.io/webcomponents/spec/shadow/)! Thanks to which we can address all those needs.


## The case

Just to specify what we are talking about, consider following scenario.

You are writing a single page app (SPA), therefore you have a place to which you would like to stamp a part of the page in run-time (include a partial).
It may happen that you are not entirely in control of what will be stamped/included. Let's say because it's third party code, you are running portal-like app, it's your code - but it's also used in other places, and you want to keep semantic structure untouched.

Take, such code as example:

```html
<html>
	... <!-- our app code goes here -->
	<div id="my-insertion-point"><!-- this is the container / insertion point, which is still under your control -->
		<your-element-0>...</your-element-0>
		<your-element-1>...</your-element-1>
		<!-- Here are the elements that were stamped from outside, for example from imported-template -->
		<foreign-element-2>...</foreign-element-2>
		<foreign-element-3>...</foreign-element-3>
		<foreign-element-4>...</foreign-element-4>
		...
	</div>
	... <!-- our app code goes here -->
</html>
```


However, it's your app and your insertion point, so you would like to make it look pretty and consistent, and you don't like to constraint yourself to CSS styles only, but use everything HTML gives you. For example build few `div`s around, add a label here or there, or maybe even re-order.


Let's say you would like to render it that way:
```html
<html>
	... 
	<div id="my-insertion-point">
		<style>
			...
		</style>
		<div class="my-class">
			<your-element-0>...</your-element-0>
		</div>
		<div class="another-class">
			<your-element-1>...</your-element-1>
			<div>
				<foreign-element-3>...</foreign-element-3>
			</div>
			<div>
				<h3>Element between</h3>
				<foreign-element-2>...</foreign-element-2>
				<h3>Element between</h3>
				<foreign-element-4>...</foreign-element-4>
			</div>
		</div>
		...
	</div>
	...
</html>
```

But still without moving actual elements in DOM tree to make any interactions work as initially, to avoid blinking and DOM manipulations overhead.


## The solution

..is simple - use [Shadow DOM](https://w3c.github.io/webcomponents/spec/shadow/). 
> You can learn more on how to use Shadow DOM for example from [Eric's article](https://developers.google.com/web/fundamentals/primers/shadowdom/)



> **tl;td** - obviously "there is a Custom Element for that" [juicy-composition](https://github.com/Juicy/juicy-composition)

Just prepare your composition as Document Fragment. 
You can programaticaly create it in JS with `document.createDocumentFragment()`, or use template element to define it more declaratively in HTML.

```html
<template id="my-composition">
	<style>
		...
	</style>
	<div class="my-class">
		<slot name="element-0"></slot><!-- for your-element-0 -->
	</div>
	<div class="another-class">
		<slot name="element-1"></slot><!-- for your-element-1 -->
		<div>
			<slot name="element-3"></slot><!-- for foreign-element-3 -->
		</div>
		<div>
			<h3>Element between</h3>
			<slot name="element-2"></slot><!-- for foreign-element-2 -->
			<h3>Element between</h3>
			<slot name="element-4"></slot><!-- for foreign-element-4 -->
		</div>
	</div>
	<slot></slot><!-- "default" slot for anything else ... -->
</template>
```
> Side note: to use it with Shadow DOM v0, you need to replace `<slot name="blah">` with `<content select='[slot="blah"]'>`

Then attach it to your container's / insertion point's shadow root:

```js
const compositionInstance = document.querySelector('#my-composition').content.cloneNode(true); // your DocumentFragment
const container = document.querySelector('#my-insertion-point');
const containersShadow = container.attachShadow({mode: open}); // new Shadow DOM v1 syntax
// const containersShadow = container.createShadowRoot(); // old Shadow DOM v0 syntax
containersShadow.appendChild(compositionInstance);
```

And it will do the job.. eventually.

There is one tricky bit remaining: we need to assign elements to slots.
Unfortunately, as there is one issue not yet solved in spec - https://github.com/w3c/webcomponents/issues/343. We need to do so by manually setting the `slot` attributes to the elements.

```js
function assignSlots(nodesList){
    for (var nodeNo = 0, len = nodesList.length; nodeNo < len; nodeNo++) {
        child = nodesList[nodeNo];
        // if element does not has explicit slot name, use 'element-{index}'
        child.hasAttribute('slot') || child.setAttribute('slot', 'element-' + nodeNo);
    }
}
assignSlots(container.children);
```
And that's it.

To react on any future changes to DOM you can wrap it with `MutationObserver`.


## Benefits

Thanks to all above:
- imported elements' structure is untouched,
- .. that's why any connections, behavior, data binding  should work as before,
- your code is clean ans semantic,
- you can use all the HTML features to apply your layout:
 - CSS styles,
 - nested HTML (Custom) Elements,
 - even scripts.
- You do not need any performance heavy JS library to materialize your virtual DOM, as it is just content distribution handled natively by browser.


If you have any comments, questions, interesting use-cases, please contact me via comments here, [twitter @tomalecpl](https://twitter.com/tomalecpl) or any applicable github repo.

### Additional resources

 - Custom Element that covers all these features: [`juicy-composition`](https://github.com/Juicy/juicy-composition)
 - Article on "HTML partials/includes WebComponents-way" [starcounter.io/html-partialsincludes-webcomponents-way/](http://starcounter.io/html-partialsincludes-webcomponents-way/)
 - Custom Element that stamps templates with partials imported via HTML Imports: [`imported-template`](https://github.com/Juicy/imported-template)
 - In-memory application platform that supports such partials and layout compositions [Starcounter](http://starcounter.com/)

