Layout compositions for HTML partials
---

Most of you are probably familiar with the concept of HTML Partial and with the way we [use them in Starcounter](http://starcounter.io/guides/web/partials/). So, you  are already aware that you can expose individual app UI as HTML Elements to be blended together with other apps and stamped to the page. Those elements could then be styled, re-arranged and composed to please the end user with more robust aesthetics and usability. In this article I'd try to show you how easily you could prepare "default" composition, so the end user will see your app elements as you desire, how to prepare the elements to look good composed with another app, and still keep you app extremely composable and flexible, so the designer would be able to do almost everything when he/she will need to prepare layout for the full suite of many apps running together.


<!--more-->

## Introduction

First let's make sure, we are all on the same page, and we know what we will be talking about.

Somewhere in your page you would like to insert a partial, you do so by simply adding [`<starcounter-include>`](https://github.com/Starcounter/starcounter-include) element to your markup:

```html
...
<h1>Some app code</h1>
...
<starcounter-include partial="{{model.SubPage}}"></starcounter-include>
```

Let's say your SubPage HTML looks like:

```html
<link rel="import" href="/some/dependency.html"/>
<template>
	<h2>Sub Page</h2>
	<template is="dom-bind">
		<p>Something bound with Polymer's dom-bind</p>
		<p>So you can use <span>{{mustaches}}</span></p>
	</template>
	<my-custom-element>...</my-custom-element>
</template>
```

`<starcounter-include>` in runtime will stamp the content of your template to itself. Therefore, you will end up with such structure:
```html
...
<h1>Some app code</h1>
...
<starcounter-include partial="{{model.SubPage}}">
	<!-- Elements stamped from your partial -->
	<h2>Sub Page</h2>
	<!-- Elements stamped from Polymer's dom-bind from your template -->
	<p>Something bound with Polymer's dom-bind</p>
	<p>So you can use <span>Polymer data-binding</span></p>
	<template is="dom-bind">
		<p>Something bound with Polymer's dom-bind</p>
		<p>So you can use <span>{{mustaches}}</span></p>
	</template>
	<my-custom-element>...</my-custom-element>

	<!-- Here other apps' elements could get stamped -->
	<other-app-element>...</other-app-element>
	...
</starcounter-include>
```



Naturally, you should attach some default styles to all the custom elements you need.

You can also use regular stylesheets to apply styles to your native and custom elements. However, if you apply specific styles to your root elements they may become hard to mix and fit in some other app.


## Layout composition and `slot` attribute

In Starcounter we really love fresh tech, so we use [Shadow DOM](https://www.w3.org/TR/shadow-dom/) to solve such problems.

It really is simpler than you may think.

> If you want to read about how it works internally take a look at [starcounter.io/unobtrusive-styling-composing-3rd-party-html-content/](http://starcounter.io/unobtrusive-styling-composing-3rd-party-html-content/)


The composition of elements is conceptually related to the insertion point and the context in which it's inserted.
That's why we allow developers and designers to apply layout composition to  `<starcounter-include>`.

You can use anything HTML gives you to build a HTML composition with slots for the content to be stamped.
We will attach this composition to Shadow Root of partial insertion point, and apps' content will be distributed as usual with Shadow DOM.

This is why, we start to advice (but NOT require) app developers to specify a `slot` attribute for their elements, with some meaningful label that can be later used to distribute it within layout composition.

> Please, take a look at [#Shadow-DOM-V1-vs-V0] section until Shadow DOM v1 is released.

### Example

Consider, you are making an app that uses a partial with
**PersonProfile.html**
```html
<template>
	<h2 slot="AnApp/header">Profile Page</h2>
	<!-- if you provide slot name it would serve as meaningful hint for composition designer, and will help to preserve layout when your partial changes -->
	<input slot="AnApp/firstname"/>
	<p>Some description</p>
	<!-- we do not require slot attribute, you will see how we handle such elements later> -->
</template>
```

Another app could map to the same partial with for example image of that person.
**Image.html**
```html
<template>
	<img slot="AnotherApp/image" src="/path/to/image.png"/>
</template>
```

Then if the designer would like to compose them together, he/she can do it via providing markup:

```html
<slot name="AnApp/header"></slot>
<!-- thanks to explicit slot name given to the element in partial,
we can refer to it in readable way -->
<div style="display: flex;">
	<slot name="AnotherApp/image"></slot>
	<div style="border: 1px solid #F7F7F7; border-radius: 3px; margin: 1em; padding 1em; display: flex;">
		<slot name="AnApp/firstname"></slot>
		<slot name="AnApp/2"></slot>
		<!-- For elements w/o explicit slot names, starcounter-include will create those.
		So, designers can distribute and style any element separately -->
	</div>
</div>
```

Such layout gives complete freedom to the designers, as they can use anything HTML allows to prepare beautiful layouts, without touching application code. Among others they can use:
 - composed HTML (custom) elements,
 - inline styles, as well as stylesheets - `<style>` tags,
 - `<link>`s with imports for required custom elements,
 - Custom texts,
 - and even `<script>`s!

Then, to insert any sophisticated, business-related element delivered by app, they simply put `<slot>` element where needed.


Compositions are stored and served from `Starcounter.HTMLComposition`. Therefore, you can:
- import/export/deploy your layouts with regular Starcounter tools,
- Use Launcher or a Layout App for live editing it browser (with code- or WYSIWYG editor)

## Default composition, standalone mode

At Starcounter we really like making things simple. That's why in case you would like to on one hand make your app mixable and blendable with other, but still make it look eye-candy from the first run, without any need of data migration or deployment. We are reducing glue-code as well as glue-work.

When you provide any set of HTML Elements in your partial (making them mixable) you already know how you would like to present them - when running standalone, or mixed with other app, without any specific cross-app layout given.

In Starcounter you can specify such default layout just along with exposed elements - in your partial. Only wrap it within `<template is="starcounter-composition">` - so it won't get rendered by browser, and we will know what to pick.

For example,
```html
<link rel="import" href="/some/dependency.html"/>
<template>
	<h2 slot="AnApp/header">Sub Page</h2>
	<template is="dom-bind">
		<p slot="AnApp/desc">Something bound with Polymer's dom-bind. So you can use <span>{{mustaches}}</span></p>
	</template>
	<my-custom-element slot="AnApp/element">...</my-custom-element>
	<template is="starcounter-composition">
		<style>
			.AnApp-article{
				border: 1px solid #F7F7F7;
				border-radius: 3px;
				margin: 1em;
				padding 1em;
			}
		</style>
		<div style="border-radius: 3px; background-color: #F7F7F7;">
			<slot name="AnApp/header"></slot>
		</div>
		<div class="AnApp-article"><slot name="AnApp/desc"></slot></div>
		<div class="AnApp-article"><slot name="AnApp/element"></slot></div>
	</template>
</template>
```

Thanks to that you can use complicated HTML & CSS structure to present your elements, and still expose individual elements for mixing and blending.

Please prefix you class names/ids within composition. It's used in Shadow DOM so it will be encapsulated out of entire page, but - as you will read later - other apps' default layouts may get inserted to the same Shadow Root.

## Merged layouts

Entire mixing and blending is about running multiple apps, so whot compositions will behave then?

The flow is simple:
1. If there is a specific composition (`HTMLComposition`) stored in data base for partial in question - we will serve it.
2. If not, we will pick default compositions from every partial and just concatenate them,
 - if there is an app that does not provide default composition, we will use just full list of applicable slots (`<slot name="App/0"></slot><slot name="App/1"></slot>...`). By full we mean - a slot for each element that is ther on first load.

If the designer starts editing of partial's layout, we provide merged default as initial one.

## Explicit vs. implicit slot names

As you have already noticed we support explicitly given slot names and we also can implicitly provide ones for you.
When should you use which one?

Implicit ones are ment to shorten code and simplify writing your first app. Even if you are not aware of mixing, blending, layouts and partials, your markup will work as you expect. Moreover, layout designers would also be able to compose your app's UI with another app's one.

But, if you are are aware of layouts, and you would like to use it by yourself or prepare your app for easier and more reliable styling - you should use explicit slot names(`slot="AppName/identifier"`).

If you are creating your own (default) composition, it will let you use human readable names instead of just numbers.
Explicit slot names plays also very important role in preserving layout when your app changes. Imagine the case, when you have made an app with few elements: a map, an input control and a button. Later, designers prepared some layout compositions. For example, to make map take entire available space and make input with button hover on top of it.

If you don't provide explicit names, designers would use numeric ones. Then, if you would like to reorder the elements or add another element - let's say header, before all those elements - all the layouts will get corrupted.
Header - the first element- will take entire space and the map will be just small box on top of it.

The simple way to avoid such problems is to specify the names - designers will use them, so the map will be always distributed where desired, and the new, non-distributed elements will appear at the bottom.

Please note, that explicit slot names, helps you handle the layout when the elements are added in run-time. Layout may describe a place for it before it's loaded.


## Where to put my styles?

In HTML you can provide CSS styles in many ways: inline attribute, via `<style>` element (inline or external), you can use UserAgant's ones, or provide some defaults for Custom Elements / style your elements' internals in their Shadow DOM.
And now you have layout compositions, and your defaults. At the first sight it may be hard to distinguish where to put your CSS rules.

First of all we are not extending nor limiting what Web Platform / browser gives you. We are just suggesting the convention to structure it a little, and providing simple way to use Shadow DOM.


Thanks to overall Web Components approach you already know that you can modularize your code and styles. If you have a CSS rule to be written somewhere, just think of the list of styling contexts that you have in such scenario, and pick applicable one

- individual Custom Element -
 the elements that used simply anywhere in your app. If the rule applies to all `<my-elements>` you should add it to element's definition / Shadow DOM.
- any layout within the root element in your partial - the element you would like to expose for blending -
 style it with regular styles (inline/external) in light DOM, as you would do in monolithic app. (Just remember to prefix global class names and ids you use in Light DOM with your app name, as it shares the scope with other apps running on entire page, see [link to Starcounter.io page])
- layout, composition of the exposed elements -
 use Shadow DOM of the partial - `<template is="starcounter-composition">` put it as inline style attribute or stylesheets there.

# Shadow DOM V1 vs V0

All above uses Shadow DOM v1 - the new version of spec that soon will be supported across all browsers.
However, as of today V1 is not yet supported by stable version of polyfills, so Starcounter still uses V0.

The only difference, which affects you as a app developer is the fact until V1 you should use `<content>` element instead of `<slot>`. So, you precisely instead of writing `<slot name="blah">` you need to write `<content select='[slot="blah"]'>`



### Additional resources

 - Custom Element for partial insertion point [`starcounter-include`](https://github.com/Starcounter/starcounter-include)
 - Custom Element that attached Document Fragment to Shadow DOM, and take care of slots: [`juicy-composition`](https://github.com/Juicy/juicy-composition)
 - Article on how Shadow DOM composition works [starcounter.io/unobtrusive-styling-composing-3rd-party-html-content/](http://starcounter.io/unobtrusive-styling-composing-3rd-party-html-content/)
 - Article on "HTML partials/includes WebComponents-way" [starcounter.io/html-partialsincludes-webcomponents-way/](http://starcounter.io/html-partialsincludes-webcomponents-way/)
