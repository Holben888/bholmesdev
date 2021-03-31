---
title: How ES Modules have redefined web development
description: The world has been moving away from "pancake" bundling and towards dynamic module imports. Let's see the latest trends and what they mean for us.
layout: blog-post
image: https://bholmes.dev/assets/blog/how-es-modules-have-redefined-web-development.jpg
publishedOn: 03-31-2021
---

You know the inspirational phrase "skate to where the puck is going?" Well, in web development... it feels like the puck is teleporting across the rink at Mach 30 sometimes.

That's how I felt diving into how ES Modules work. Turns out, there's been some huge shifts right under my framework-laden nose these past few years. After discovering that this is _valid JS_ [across all major browsers](https://caniuse.com/?search=dynamic%20import)...

```js
const js = await import('script.js')
```

...I had to make a post about it. So let's explore

1. 🥞 My misconceptions about what bundlers do these days
2. 🧩 What ES Modules + dynamic imports can do
3. 🚀 How build tools are evolving for the post-IE era

_**Onwards!**_

## Foreward: Personal delusions in a webpack world

What I'm sharing here is probably common knowledge to some. Heck, import-able JavaScript modules have lurked in the [ECMAScript](https://stackoverflow.com/a/33748435) standard [since 2017](https://medium.com/@giltayar/native-es-modules-ready-for-prime-time-87c64d294d3c)! But if you've been using "traditional" project configs like `create-react-app` for a long time, you might think that old-school bundling is how the world works.

So let me _ahem_ unpack the traditional definition of "bundling." In short, it's the concept of taking a chain of JS files like this:

```js
// toppings.js
export {
  blueberries: async () => await fetch('fresh-from-the-farm'),
  syrup = "maple",
}

// ingredients.js
export { flour: 'white', eggs: 'free-range', milk: '2%', butter: 'dairy-free' }

// pancake.js
import { blueberries, syrup } from './toppings'
import { flour, eggs, milk, butter } from './ingredients'

const pancake = new Pancake()

pancake.mixItUp([ flour, eggs, milk, butter ])
pancake.cook()
pancake.applyToppings([blueberries, syrup])
```

And "flattening" the import / export chains into a big bundle pancake 🥞

```js
// bundler-output-alksdfjfsadlf.js
const toppings__chunk = {
  blueberries: async () => await fetch('fresh-from-the-farm'),
  syrup = "maple",
}

const ingredients__chunk = { flour: 'white', eggs: 'free-range', milk: '2%', butter: 'dairy-free' }

const { blueberries, syrup } = toppings__chunk
const { flour, eggs, milk, butter } = ingredients__chunk
const pancake = new Pancake()

pancake.mixItUp([ flour, eggs, milk, butter ])
pancake.cook()
pancake.applyToppings([blueberries, syrup])
```

So we're compressing all the JavaScript files we're developing into a _single_ file for the browser to consume. Back in 2015-era web development, this really was the only way to pull off "importing" one JS file into another. `import` wasn't even valid JavaScript! It was just some neat trickery that build tools like webpack could pick up and understand.

But silently, in the depths of the ES spec, `import` and `export` syntax _did_ become valid JavaScript. Almost overnight, it became feasible to leave all your `import` and `export` statements in your code or even _gasp_ ditch your JS bundler entirely 😨

This innovation became what we call **modules.**

## ES Modules

There's an [in-depth article from MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules#other_differences_between_modules_and_standard_scripts) on this topic that's _well_ worth the read. But in short, "ES Modules" (sometimes denoted by [`.mjs` files](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules#aside_—_.mjs_versus_.js)) are JavaScript files with some exported values for others to import and use. As long as you load your "entry" files with the `type="module"` attribute:

```html
<script type="module" src="pancake.js"></script>
```

That file is ready to `import` all the other scripts it wants! Well, as long as those other scripts exist in your project's build of course (we'll ignore CORS issues for now 😁).

This concept of importing what's needed over "flattening all the things" has some nice benefits:

1. **You don't need to load and parse _everything_ up front.** By default, anything `import`ed is ["deferred" for loading as needed](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules#other_differences_between_modules_and_standard_scripts). In other words, your computer won't turn into a fighter jet trying to load JS when you first visit your website.
2. **The need for tooling like webpack can (one day) disappear ✨** Bringing browsers closure to how _humans_ write their code is a huge win for newbies and pros alike 🏆

### Dynamic imports take it a step further

Dynamic imports are the spicier side of ES Modules that _really_ make things interesting. As [this article from the V8 team](https://v8.dev/features/dynamic-import) describes (creators of Google Chrome's rendering engine), a **dynamic import** is an asynchronous fetch for some JavaScript whenever you need it.

It's very similar to the [`fetch` API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) in a way! But instread of grabbing some JSON or plain text, we're grabbing some real, _executable_ code that we want to run.

All you need is a humble one-liner:

```js
const lookAtTheTime = await import('./fashionably-late.js')
```

...and you just grabbed all the `export`s from that file. Loading JS on-the-fly like this has a _ton_ of benefits if you're working with [single page apps](https://www.bloomreach.com/en/blog/2018/07/what-is-a-single-page-application.html) like NextJS or `create-react-app`. The V8 team offered this elegantly simple take on client-side routing, only loading the JS you need when you click on a link:

```js
const links = document.querySelectorAll('nav > a');
for (const link of links) {
  link.addEventListener('click', async (event) => {
    try {
      // go grab whatever JS the route may need
      const module = await import(`${event.target.href}/script.mjs`);
      // The module exports a function named `loadPageInto`,
      // Which might render some HTML into the body
      module.loadPageInto(document.body);
    } catch (error) {
      document.body.innerHTML = `
        <p>404 page not found</p>
      `
    }
  });
}
```

**I basically just implemented a router in 10 lines of code.** (yes, that's a _serious_ overstatement, but it's closer than you might think).

This falls into [code splitting](https://developer.mozilla.org/en-US/docs/Glossary/Code_splitting), aka loading "components" (or modules) of code whenever the user needs them. Back in the dark ages of bundle all-the-things, you'd have to load all these components up front. This could mean _thousands_ of lines of dead code!

## So wait, it's 2021... why does all my tooling look the same?

This was certainly my first question when I read up on this. I recently graduated from [`create-react-app`](https://create-react-app.dev) to [NextJS](https://nextjs.org) as my React boilerplate go-to, but there's still that same webpack configuration + bundle process to think about 🤷‍♀️

A lot of this is just the curse of abstraction. Looking under the hood, these tools have made _great_ strides since ES modules hit the scene. Namely, tools like NextJS can magically "split" your React app into bite-sized chunks that get loaded as-needed. This means:

- only load the JS for a page **when you actually visit that page**
- only load React components **when they actually need to display**
- (bonus) pre-fetch JS **when someone is _likely_ to need it.** This is a more advanced feature ([documented here](https://nextjs.org/docs/api-reference/next/link#if-the-route-has-dynamic-segments)), but it lets you do all sorts of craziness; say, grabbing resources for a page when you hover over  link

There's also the benefit of **backwards compatibility** when using a bundler. For instance, Internet Explorer has no concept of "modules" or "import" statements, so any attempt to code split will blow up in your face 😬 But with a meta-framework like NextJS by your side, you can polyfill such use cases without having to think about it.

## Approaching the post-IE age

If you haven't heard, a major announcement sent ripples through the web dev community recently: **[Microsoft will officially drop IE 11 support for its products in August 2021](https://techcommunity.microsoft.com/t5/microsoft-365-blog/microsoft-365-apps-say-farewell-to-internet-explorer-11-and/ba-p/1591666)** 😱

Many are treating this as the ticking timebomb for legacy browser support. When it goes off... we might be safe to lose our polyfills for good. Yes, certain sites for governments and internal business operations will probably stick to their PHP-laced guns. But for us bleeding-edge developers, we may have a whole new frontier to explore 🚀

### A world of bundlers that... don't bundle

The tides have certainly shifted in the JS bundler community in the past year. With the prospect of dropping polyfills and aggressive bundling for good, people started turning to the _real_ reasons you want a bundler:

- **To process all your fanciness that _isn't_ valid JS.** Think JSX for React components, TypeScript for type checking, Styled Components and CSS modules for CSS-in-JS, etc etc.
- **To spin up your app locally.** You could always open HTML files in your browser directly, but you'll loose all that immediate feedback! You should see all your new JS and CSS the millisecond you hit "save."
- **To optimize code for production.** You'll probably want some last-minute stripping for added speed, like removing `console.log`s, minifying everything, linting, and so on.

Because of this refined feature set, the new wave of JS processors are just calling themselves "build tools" to stay more generalized.

[**Snowpack**](https://www.snowpack.dev) is really what got the ball rolling from my perspective. They promise all the selling points I listed above, plus the absolute fastest live-reloading in the biz. This is mainly because of that code splitting I mentioned earlier. Since they leave all those modules and dynamic imports in-tact, **they avoid re-processing the JavaScript that didn't change.** So if you just updated a single React component, it'll reprocess those 10 lines of code and blast it onto the page in a flash ⚡️

[**Vite**](https://github.com/vitejs/vite) is a major contender to note as well. This one was spearheaded by [Evan You](https://twitter.com/youyuxi?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor) (the overlord of VueJS) to tackle a similar feature set to Snowpack. It's far too early to say whether I'd _prefer_ this setup to Snowpack, but [here's a nice comparison piece](https://blog.logrocket.com/vite-vs-snowpack-a-comparison-of-frontend-build-tools/) if you're considering either for serious apps.

There's also the crazy world of using different programming languages to process your code. [ESBuild](https://github.com/evanw/esbuild) is a big contender right now, using GoLang to process JavaScript in no time flat.

### Call to action: explore these new build tools!

It's definitely worth your time to whip up a sandbox and start compiling some code. [Vite's `create-app` tool](https://github.com/vitejs/vite/tree/main/packages/create-app) is a great one for it's beginner friendliness, with options to use any major framework out-of-the-box (React, Vue, Svelte, and even [Lit Element](https://lit-element.polymer-project.org)!).

I was caught off guard to find **there's no build directory** when working in development. The code your write gets mapped to the browser directly, processed on-the-fly whenever you save ❤️

So go forth and see what the future is like! With any luck, we'll get to have our bundle pancake and eat it too 😁
