---
title: Want CSS variables in media query declarations? Try this!
---

If you're like me, you've probably been [stretching CSS variables / custom properties to their limits](https://bholmes.dev/blog/how-using-css-variables-cut-down-on-my-javascript/) while building your own design systems. But this "silver bullet" can lead to a nasty roadblock: **you can't use them in media query declarations.**

To clarify, this is the behavior you might be after:

```css
:root {
	--mobile-breakpoint: 600px;
	--tablet-breakpoint: 900px;
}

@media(max-width: var(--mobile-breakpoint)) {
	.obligatory-hamburger-menu { visibility: visible }
}
```

...which sadly won't work.

## Why var() can't work in a @media declaration

This may seem confusing at first. If our variables are available at the `:root` of the page, why can't a media query access them?

Well, it comes down to what the `:root` element actually means: **the root element of the HTML document.** But conceptually, media queries aren't attached to HTML elements at all. These declarations are processed while your CSS is being parsed, so it won't know to look to the `:root` and pull in the variable values.

If this seems confusing, let's consider a CSS variable like this one on the `body` element:

```css
body {
	--mobile-breakpoint: 600px;
}
@media(max-width: var(--mobile-breakpoint)) {...}
```

Clearly, our `@media` query doesn't know about this `--mobile-breakpoint` variable since it doesn't "belong" to the `body` element. Heck, it doesn't belong to any element in your HTML, which explains why our `:root` solution won't work either.

Sass / SASS can make this even more confusing by allowing `@media` blocks to be "nested" inside other rulesets. Don't be fooled! When your lovely Sass gets compiled, those media queries float right back out to the base of your CSS document.

### Hope on the horizon for CSS "environment" variables

As luck would have it, the W3C isn't happy about this limitation either. [Their proposal](https://drafts.csswg.org/css-env-1/) for "environment" variables is at the _earliest stage_ right now (phase 1 as of May 2021), but it seems to tackle this very issue!

I'll avoid showing you the syntax since it'll likely change overtime. Just know it's meant to address variables that exist _beyond_ the HTML element level for use cases like `@media` declarations.

## Workarounds you can try

If you want media query variables and want them _now,_ there's a few options available to you.

### For Sass / SCSS users

The first is what I'd recommend to existing Sass users: **fall back to Sass variables for media queries, and use CSS variables everywhere else.** 

This is because, unlike CSS variables, Sass variables don't "belong" to CSS rulesets under-the-hood; They're just a piece of syntactical sugar for SASS to pick up and "paste" in the correct value. Just know that you'll need to fall back to Sass for math calculations as well instead of using the `calc()` function (because this isn't valid either!)

```scss
/* ✅ good */
@media(max-width: $mobile-breakpoint + 50px) {...}
/* ❌ bad */
@media(max-width: calc($mobile-breakpoint + 50px)) {...}
```

### For anyone else

If you either don't use Sass or don't want 2 different syntaxes for your variables, there's a [neat PostCSS plugin](https://www.npmjs.com/package/postcss-media-variables) you can try too. Just tack this onto an existing CSS build step (or introduce a build step against your will 😣) and _BAM!_ CSS variables _and_ the CSS `calc()` function will work in your media query declarations 👍

⚠️ **Word of caution** on this solution though. You'll be writing _invalid_ CSS by current browser standards and making it valid, which can look pretty confusing to new codebase contributors. At least be sure to document this plugin in your project README if you decide to go this route.