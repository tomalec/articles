`<starcounter-include>`@3.0.0 & changes required in views
---

With latest Release Candidate Starcounter 2.3.1.7022 we will ship the new version of `<starcounter-include>` - the element you use to include one (blended) partial view in another. It not only provides set of bug fixes, but it also makes us closer to the platform, Web Components V1 spec and removes confusing magic that was automatically creating slots.

In this article I would like to guide you through those changes, so you will know what behavior to expect, what code to change and how. Fortunately, most of those changes could be prepared in advance as they will not break existing behavior.

<!--more-->

I assume, you already have some knowledge on client-side blending and you have some partial views already running.
If not, feel free to go through [updated docs](https://docs.starcounter.io/guides/web-apps/html-view-guidelines/)

## TL;DR

The main changes are:

1. `<juicy-composition>` was removed -

    previously your content was stamped at `starcounter-include > juicy-composition` now it's stamped directly in `starcounter-include` so your CSS selectors become shorter, you don't have to bother about additional wrapper and you are able to select and style nested `starcounter-include` from parent one's Shadow DOM.
2. specific `<slot name="...">` elements are no longer automatically created for every Light DOM element -

    that improves the performance heavily, removes confusing magic, makes us closer to just native Shadow DOM behavior - it's not just declarative, allows you to hide your element in Shadow DOM layout composition by just removing its `slot`.
3. implicit slot names (like `slot="myapp/1"`) are no longer automatically created -

    again it increases performance, reduces the magic. Also, it's requirement of the point above.

    In SD V1, if element has a `slot` attribute attached it **will NOT** be distributed by default slot `<slot></slot>`. Therefore if we would attach `slot` attributes without creating `<slot name="_">` elements, we would hide all elements by default.
4. Default slot (`<slot></slot>`) is now served from server-side for default `declarative-shadow-dom` -

    don't add it by yourself.
5. `<starcounter-include>` is Custom Element v1 -

   it should run faster in browsers that support it (all major ones, soon). And given [`document-register-element` polyfill](https://starcounter.io/week-starcounter-30062017/) it makes us closer to be fully migrated to WC v1.

### `<juicy-composition>` was removed

All the functionality is now covered by the `<starcounter-include>` (except slots magic - [2.](#) & [3.](#)).
It handles `declarative-shadow-dom`, it fetches specific layout compositions from the database, but it no longer wraps your elements in Light DOM within yet another element.

This gives you not only cleaner and shorter code, but given Shadow DOM V1 constraints, will finally let you select and style container of elements stamped from nested `<starcounter-include>`. For example:

**ParentPage.html**:
```html
<template>
	<h1 slot="myapp/parent-header">I'm parent page</h1>
	<starcounter-include slot="myapp/parent-details"></starcounter-include><!-- loads SubPage.html -->
	<!-- This could come from serverside as well -->
	<template is="declarative-shadow-dom">
		<style>
		/**
		 * Make nested `<starcounter-include>` be a CSS Grid,
		 * and display SubPage.html elements in this grid.
		 * Previously it was selecting only a wrapper of actual container.
		 */
		#my-app-details-grid ::slotted(starcounter-include){
			display: grid;
		}
		</style>
		<slot name="myapp/parent-header"></slot>
		<slot id="my-app-details-grid" name="myapp/parent-details"></slot>
	</template>		
</template>
```

Naturally, this change may affect your existing CSS selectors.

✏️ All you need to do is to review all your CSS (& JS) rules that use `juicy-composition` or `starcounter-include` in selectors. Usually, you would need just to remove `juicy-composition` part, but this may depend on your actual CSS rules.



### Specific `<slot>` elements are no longer automatically created

Previously, even if you didn't provide any `declarative-shadow-dom` or specific layout composition in the database, or simply missed few `<slot name="...">` elements, we were creating them automatically for you.

This had significant cost:
- we had to make all those checks and updates in run-time on every DOM mutation, which is simply expensive,
- such automation magic was sometimes confusing, for those who expected `<starcounter-include>` just to use given layout composition in Shadow DOM,
- to hide/don't distribute an element you had to wrap it in a hidden `<div>`, what was un-intuitive and hacky,
- it may result in <abbr title="Flash of Unstyled Content">FOUC</abbr> in polyfilled browsers or some border cases.

That's why now, we no longer do so.

✏️ If you want your elements to be blendable and be able to distribute them somewhere else than in default `<slot>`, you need to explicitly add `slot` attribute to your element and respective `<slot ...>` element in your `declarative-shadow-dom` and/or in specific layout composition in the DB.

This may require a little bit more boilerplate code for blendable apps, but on the other hand, everything becomes clear, explicit and intuitive. Now to hide/don't distribute a `<div slot="myapp/my-name">` all you need to do is to remove `<slot name="myapp/my-name">` from your composition.

### Implicit slot names (like `slot="myapp/1"`) are no longer automatically created

Previously if you forgot or omitted to add an explicit `slot` attribute/name to your Light DOM element, we were generating it automatically, so it could be blended later as an individual `<slot name="...">` (insertion point).

Once again it was not for free:
- it was expensive, problematic and bug prone to generate in as the elements were added and removed dynamically at run-time,
- it was confusing to developers when and why those names appear,
- as Shadow DOM is relatively new technology, adding non-standard magic to it was leading to lots of misunderstanding,
- updating the app view could invalidate and break existing layouts in a very mysterious way,
- given the way SD v1 handles default slot, adding a slot name without `<slot ...>` element (see [above](#)) would hide it.
  In V0 default slot `<content></content>` was distributing all **not yet distributed** elements, in V1 default slot `<slot></slot>` distributes all **not assigned** elements.

Auto slot names and elements were pretty handy when we were migrating from old strict `juicy-tile-grid` solution to complete freedom of Shadow DOM layouts. But now they seem to do more harm than good. We would like to continue with an approach closer to Web Platform.

If you miss those features please let us know we may think of reimplementing those sugars later, once we are migrated and feel cozy in V1 environment - hopefully in less expensive manner.

Due to how default slots behave in V1, if element has no `slot` attribute at all it still will be distributed, but this time in default slot (`<slot></slot>`) bulked with other not assigned elements, with no way to distribute separately afterwards - for example when preparing specific layout for combination of apps.

✏️ For your existing views, this change means you need to add `slot` attributes (and elements) to all your Light DOM elements that you want to to be able to blend separately in future. Usually it means all visual elements, `<style>, <script>, <meta>, <template>` work fine if not distributed anywhere or put in default slot.

### Default slot (`<slot></slot>`) is now served from serverside for default `declarative-shadow-dom`

Back then, when we were creating `<slot ...>` elements for every (not-yet-distributed) Light DOM element, there was no need for default `<slot>`.
Now it's useful and required.

Since we discourage to add it manually in `declarative-shadow-dom` in any app - as you may hijack other apps' elements, we will create it automatically and append it to the bottom of the merged view. So, all not assigned elements from all the apps will be by default distributed at the very bottom of each view.

This could be changed in specific layout composition stored in DB. Then, when you are already aware of all the apps running, you are free to place it anywhere you want, or event not placed it at all.

✏️ For this change, you need no action. The result will look exactly the same as before - default `<slot>` is on the bottom in the place where all automatically created `<slot ...>` elements were placed. Just for the new specific compositions make sure you have it there (if you want it, naturally).

### `<starcounter-include>` is Custom Element v1

That's the thing which should not affect you anyhow, except the fact you can now feel closer to fresh and shiny Web Components V1.

Just make sure your environmant is capable to run Custom Elements v1. We [added CE v1 polyfill](https://starcounter.io/week-starcounter-30062017/) to Starcounter's `/sys` folder and `PartialToStandaloneHtmlProvider`.
If you have overwritten it, make sure to add `<script src="/sys/document-register-element/build/document-register-element.js"></script>`.

🎉 This allows you to run Custom Elements V1 together with V0 and migrate gracefully.

## Summary

As you can see, once again we are trying to bring you closer to the Web Platform and collapse the stack.

We hope that by removing all unnecessary magic, we will make our solution faster, easier to understand and more flexible.

If you still miss those features, feel unsure about migration needed, or have any other comments, please feel free to reach us.
Either by commenting here, posting an issue at GitHub, contacting us via Slack, Twitter or any other channel.


### Additional resources

 - `<starcounter-include>` release notes - [github.com/Starcounter/starcounter-include/releases](https://github.com/Starcounter/starcounter-include/releases)
 - Guide to migrate Shadow DOM v0 to v1, by W3C spec editor - [hayato.io/2016/shadowdomv1/](https://hayato.io/2016/shadowdomv1/)
 - Article on how Shadow DOM composition works [starcounter.io/unobtrusive-styling-composing-3rd-party-html-content/](http://starcounter.io/unobtrusive-styling-composing-3rd-party-html-content/)
 - Article on "HTML partials/includes WebComponents-way" [starcounter.io/html-partialsincludes-webcomponents-way/](http://starcounter.io/html-partialsincludes-webcomponents-way/)
