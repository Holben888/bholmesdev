---
title: How using CSS variables helped me cut down on JavaScript
image: https://thepracticaldev.s3.amazonaws.com/i/x586421hp4fxpcsxfdvw.png
description: Styling with JavaScript is pretty convenient, but how can CSS replicate that logic?
layout: blog-post
date: 2019-05-31T03:37:13.004Z
---

_**Note:** My post is inspired by [this article](https://css-tricks.com/keep-math-in-the-css/) from the brilliant Chris Coyier about the power of CSS `calc()`. Definitely worth a read!_

If you’ve been keeping up with web development since 2018, you’ve probably come across [CSS custom properties / variables](https://developer.mozilla.org/en-US/docs/Web/CSS/--*). They’ve become the new hotness in applications even beyond just using raw CSS, as they offer scoping and cascading that even new CSS-in-JS solutions like [Styled Components](https://www.styled-components.com/docs/basics) don’t directly replicate.

I hadn’t given CSS custom properties much of a fair shake when I first learned about them since I’ve become such a prop slinging, CSS-in-JS devotee (please hold your criticism 😛), but my recent project required me to go back to traditional stylesheets due to using the exotic framework, [SvelteJS](https://svelte.dev).

At first, simply declaring styles as necessary without any form of variables seemed manageable; whenever something custom needed to happen based on code, I could just whip up a string of inline CSS and tack it onto my element without worrying too much about optimization. However, while the website still seemed snappy, my codebase was growing less and less readable. Enter: CSS variables!

_**Sidenote:** There was a time people would shout from the rooftops "they're not called **variables**; they're **custom properties!**" Thankfully, relevant MDN documentation and general lingo has caught up to just calling them variables. So either works, but "variables" is a little clearer in my opinion_ 😊

## So how do CSS variables work?

To those unfamiliar, you can declare a variables within any element you choose, or within the `:root` selector to make it globally accessible. Just use `--[property-name]` syntax so CSS will pick it up as a variable…
```css
:root {
    --global-var: 50px;
}
.some-class {
    --scoped-var: 4%;
}
```
… and then use those variables in sub-elements (or any element for global properties) using `var(--[property-name])`.
```css
.child-of .some-class {
    margin: var(--scoped-var);
}
```
This is similar to how [CSS attributes](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors) work, except CSS variables can take on any unit of measure you choose, then be used to define any CSS property. This means you get the same kind of flexibility that variables in pre-processors have been offering for years now, though with an admittedly clunkier syntax (hey, that’s CSS3 for you 🤷‍♀).

What’s not as well known is how CSS variables can be **unitless.** This doesn’t seem like a huge deal at first, but it offers a big advantage: combined with `calc()`, CSS variables can be used to scale properties by a set amount. This was invaluable in refactoring own code, since it meant I could rewrite my CSS string construction in JavaScript with just a couple lines of CSS calculation.

## Let’s see an example

To show custom properties in action, I’ll take a logic snippet from a portfolio concept I built out.

The goal was simple: I wanted an accent bar to cycle through a set of gradients on a loop, shifting from one gradient to the next. This would be possible with a single animation keyframe, though I had a caveat: a looping interval was being used on other elements of the page with logic CSS couldn’t replicate, and I wanted to use the same interval in my accent bar for consistency's sake. This interval was, of course, defined in JavaScript using `setInterval(...)`. Whenever the callback function was hit, some CSS needed to change. This interval was set in a parent component and accessed within my accent bar component (yes, I'm using a component-based framework).

Before diving into the example, note that this project was built on Svelte. This shouldn't affect the readability of the code too badly; just accept that the lesser details involve some magic ✨

![Animated gradient of colors](https://thepracticaldev.s3.amazonaws.com/i/dlxut0dvc69sx06dzuzp.gif)

_The end goal_ 

## My old way of doing it

Originally, I cycled through the gradient by creating a wide background image with hidden overflow, then shifting the background position on each tick of the interval. This gave the illusion of the background shifting colors, though it was essentially moving through one large gradient. However, this background position required a fair amount of calculation.

To keep all the interval tracking simple across several components, I was keeping track of a  `gradientIndex` variable passed as a prop. This index corresponds to a list of gradient colors I am cycling through called `GRADIENTS`.

However, this means some extra logic is necessary to update the CSS: whenever the `gradientIndex` is changed, a new CSS string needed to be constructed to be applied as an inline style. Thus, we need to dig out a lifecycle method to construct our strings when the `gradientIndex` prop changes. In Svelte, this is accomplished using the `afterUpdate` callback function:

```js
...
afterUpdate(() => {
  backgroundPosition = `${(100 / (GRADIENTS.length - 1)) * gradientIndex}%`;
});
```

We also need to figure out the background size for our overflow by getting a percentage from `GRADIENTS.length`:

```js
const backgroundSize = `${GRADIENTS.length * 200}% 100%`;
```

Finally, we throw this into our inline style alongside our constructed linear gradient background:

```html
<span
  class="bar"
  style="background-image: {backgroundImage};
  background-position: {backgroundPosition};
  background-size: {backgroundSize}"
></span>
```

So yes, the end result functions pretty well without any performance problems... on my overpowered MacBook anyways 😛 However, we've added a fair amount of complexity that will only get worse as we scale. We've added a lifecycle method to handle our inline CSS construction, and we're littering our JavaScript with variables that would ideally be kept within the styles where they belong. If only there was a way to calculate this using just CSS!

## A new, more readable solution

So how can we tackle this problem using CSS variables? Well, looking at the string for background position constructed in JS, we see the calculation requires knowing how many gradients there are (`GRADIENTS.length`) and the current index to figure out position (`gradientIndex`). So, why not just make each of these CSS variables?

Thankfully, CSS variables are settable using inline styles like any other CSS property (the same can’t be said for variables in SASS!). So, let’s say both our aforementioned variables are part of the component state. We can make them visible to CSS using the following inline style:
```html
<span
  class="bar"
  style="background-image: {backgroundImage};
  --index: {gradientIndex};
  --length: {gradientLength}"
></span>
```
Now, we can determine our background size and position within CSS just by using `calc()`:
```css
.bar {
  --index: 0;
  --length: 0;
  background-size: calc(var(--length) * 200%) 100%;
  background-position: calc((100 / (var(--length) - 1)) * var(
  --index) * 1%);
}
```

There are a couple things to unpack here. First, we set each variable to some initial value for the sake of completeness. This isn’t necessary since the inline style should always be applied, though initializing CSS variables is a good habit to get into. Next, we set our background position similar to our JS, with one notable difference: we multiply the `--index` by a percentage, rather than write the percent sign directly after the variable. This is because `calc()` treats the variable like a constant in mathematics, so it has to be multiplied by some value for a unit of measure to be applied.

Oh, and here’s our new JS snippet:
…wait, there isn’t one anymore! 🎉

### Can we go even deeper?

Something this example doesn't take advantage of is variable cascading. This is super useful for component-based development, since you can consolidate a lot of wacky CSS calculation into the parent component. Then, child components can just access the CSS variables from higher up in the cascade. In our example, we could make `gradientIndex` a CSS variable in the parent wrapping around our color bar and avoid passing it as a prop entirely!

Of course, this can start to have a negative impact on readability, with variables several levels up cascading down without the developer realizing it. This exposes the age-old conflict between cascade thinking and component-based thinking, so use this technique sparingly.

## Wrapping up

With that, it should be clear that custom properties can be pretty powerful to take your JavaScript logic over to the stylesheet. Additionally, now that CSS variables are compatible with most all modern browsers (except IE of course 😢), they should be pretty safe to experiment with even in production code. So go forth and get to styling!
