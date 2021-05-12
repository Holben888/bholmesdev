---
title: Why SvelteJS may be the best framework for new web devs
image: https://thepracticaldev.s3.amazonaws.com/i/c3s8y2nj2safv7q58qs9.png
description: Every new programmer starts with vanilla JS and DOM manipulation. But in the component-driven world of modern web dev, is that the best way to start?
layout: blog-post
date: 2019-03-19T22:03:49.242Z
---

Any web dev who's been at it for a few years has likely heard this question every other day.

*I'm really interested in learning about web development, but I don't know how to start. Any suggestions?*

A decade ago, this would have been an easy answer. Just make an `index.html`, throw some tags in there, make that header turn red with CSS, and reach for JQuery (or plane ole JavaScript, depending on who you ask) to handle those mouse clicks!

...Oh, how things have changed. Now we're running around with build tools, client side routing, special frameworks with fancy runtime scripts, binding "this" everywhere, template literals, CSS-in-JS... how do we choose what matters most? 🤷‍♀️ We can't start teaching how React uses a virtual DOM if someone doesn't even know what the DOM is!

This has led to countless avenues to start with with varying degrees of "just type this now, and I'll explain later." Some encourage beginners to just learn React or Vue right away to get started with modern practices, while others scream from the mountaintops that **beginners should always start with the fundamentals.** Truthfully, there are merits to both approaches. The former can get newbies excited with hot reloading and components, running the risk of leaving too much to the unknown, while the latter gets beginners understanding how DOM manipulation works under the hood, while possibly steering people away with the complexities of JS we've since abstracted away.

What we need, then, is a middle ground. A way to get started with the fundamentals while soaking up modern concepts like component-driven development, scoped vs. cascading CSS, templating, declarative functions, etc etc etc.

##Enter: Svelte

SvelteJS is a pretty new kid on the block just starting to get some attention. Some may know it as the [most popular write-in for the State of JavaScript 2018](https://2018.stateofjs.com/front-end-frameworks/other-libraries/). For the abridged explanation, Svelte is  meant to be the framework that isn't really a framework; it's basically a tool to compile components down at the build step, allowing you to load a single `bundle.js` on your page to render your app. This means no virtual DOM, no frameworks on top of frameworks, and **no framework to load at runtime**.

These are pretty big selling points for more experienced developers, but most beginners probably couldn't read that last paragraph without their head exploding. Luckily, it's not this compilation sorcery that makes Svelte so beginner-friendly. It's actually the syntax.

If you've never seen a Svelte component before, here's a really basic example:

```html
<p class="pretty">Here's some markup <strong>written by {name}!</strong></p>

<style>
    /* here's some scoped CSS */
    .pretty {
        color: red;
    }
</style>

<script>
    /* ...and a variable we can access in the markup */
    let name = "Ben";
</script>
```

Let's unpack this a little. So first off, it's worth noting that all of this can live inside a regular `.html` file, or a `.svelte` file if your heart desires. Also, we see some familiar tags reminiscent of framework-less development: `<style>` and `<script>`. Sadly, writing styles and JS in these tags is necessary for properly building scoped components, but it allows syntax highlighting to work without extra text editor extensions like CSS-in-JS solutions. Plus, Svelte's build step is smart enough to scope any component-specific styles by default, so you won't have styles bleeding between components.

You'll also see some magic happening with that JavaScript variable called `name`. This is a shiny new concept for Svelte v3, where **any** variable in your component's script is accessible from markup. Thus, there's no framework specific syntax to learn for state management, so no Angular `$scope`, no React `this.state`, and no Vue `data`. Instead, we can just use `let`s everywhere to get assignable state values, cuing re-renders whenever these values change.

This freedom from this jargon means making a component can almost feel like whipping up a CodePen, but without wondering how to connect that declarative JS function you learned to some DOM query selector. Don't worry too much though: Svelte won't mess with callback functions or window listeners, so those fundamentals remain.

The other nice thing about these components is that they're just as import-able as a traditional component. Just import the `.html` and Svelte knows how to unwrap it 😊 

```html
<div>
    <Wizardry />
</div>
<script>
    import Wizardry from './wizardry.html'
</script>
```

### Neat, but hang on a minute...

Some readers may find this concept as mind-blowing as I do, but others probably have their pitchforks ready at the thought of throwing this at beginners. Won't this confuse them about how DOM manipulation really works? 

The answer is... maybe. But when someone's just starting out (at least from personal experience), it's okay to accept a little abstraction for the sake of making cool things more quickly. 

Also, just as languages like Java and JS have abstracted away pointer management with garbage collection, it feels like most every modern web development tool has abstracted away DOM manipulation, save for more advanced edge cases beginners likely won't need to face. Btw, if you are scratching your head at what pointer management is, I think that kind of proves my point 😛 Thus, rather than forcing beginners to manipulate the DOM or grasping framework-specific state wrappers, why not just let them access variables directly from markup? Now they can learn the basic principles of component state without getting caught in the weeds.

## Okay, but how much "special syntax" is there?

There's some templating to pick up here admittedly, but it's a lot less than you might think! One Svelte-y syntax is for looping and conditionals for displaying DOM elements. This works a lot like the JSX way of returning elements from a `map`, but without all the nested brackets beginners (and myself) can easily get lost in. Here's a basic one that generates a list of a elements from an array:

```html
<ul>
    {#each profiles as profile}
    <li>{profile.name}: {profile.role}</li>
    {/each}
</ul>

<script>
    const profiles = [
        {name: 'Wes Bos', role: 'React extraordinaire'},
        {name: 'Chris Coyier', role: 'Father of CodePen'},
        {name: 'Cassidy Williams', role: 'Letting you know it's pi time'}
    ]
</script>
```

Again, I understand any criticisms about the syntax, but what I love is how easily understood it is. Instead of nesting JavaScript in our HTML, we just say hey, lemme loop over this array and create an element for each one.

There's another Svelte oddity worth mentioning that I'm admittedly not as thrilled about: passing props to components. Yes, it is fully functional and easy to learn, but the syntax is a bit too magical for some people's tastes. To handle props, we simply pass the prop to the component wherever it's imported...

```html
<!-- somewhere.html -->
<Profile coolGuy="Scott Tolinski" /> 
```
...and get that variable as an exported "let"
```html
<!-- profile.html -->
<p>{coolGuy}</p>
<script>
    export let coolGuy = '';
</script>
```
I totally understand if some are turned off by abusing "export" like this, but it does at least follow the way beginners should conceptualize modules: we export what we should access outside of the component, and import what we want to show in our component. 

## What about the build process though?

Another criticism about getting beginners started with frameworks is the need to use a package manager. Which means... *gasp* using the terminal! 

Look, I get it, popping that thing open can be intimidating, with its monospace font and that spooky "cd" to jump directories. But to be fair, it's really not a huge hurdle when you only need a single command to get running. Additionally, the [integrated terminal within VS Code](https://code.visualstudio.com/docs/editor/integrated-terminal) makes it dead simple to get started with; it even plops you down in your current project directory! 

Svelte actually offers [a downloadable starting point](https://v3.svelte.technology/repl?version=3.0.0-beta.20&example=hello-world) that's nice outside of the box, but I [made my own starter template](https://github.com/Holben888/svelte-starter-template) that just uses the build tool Rollup for live reloading. In fact, the config file is under 30 lines long! For a basic Svelte project, these is all the directories and files you need:

```
/public
    index.html
/src
   index.html
app.js
rollup.config.js
package.json
```



Just add a command to run the build step in the `package.json` and you're all set! You could certainly say that all the extra modules and files other frameworks need aren't a problem if beginners don't touch them, but in my eyes, the less extra stuff for newbies to wonder about, the better. 

## But is it a stable framework?

This is a very relevant question for a framework as young as Svelte. All examples I have shown use the syntax of Svelte version 3, which ~~is still in beta as of the time of this writing~~ has a relatively small following compared to framework behemoths like ReactJS and VueJS. So as exciting as it is, I would wait another few months before rushing to teach code workshops with it. Still, Svelte offers a [really concise page for documentation](https://svelte.dev/docs) for version 3 that can ease beginners into the framework without getting overwhelmed by subpage after subpage of explanation.


So let's go over some of the main selling points for learning web development with Svelte:

- It's a component-based framework with 0 extra plugins
- It handles state management without all the usual cruft
- It uses scoped styling without needing CSS-in-JS (so no editor extensions or wacky syntax)
- It only needs a dead simple build script to get going
- Hardly any files are needed in a base project

Of course, it's totally fine if I haven't convinced you with this post; all good posts stoke a little controversy! But I hope it at least showed you how freaking cool and simple Svelte can be to learn.
