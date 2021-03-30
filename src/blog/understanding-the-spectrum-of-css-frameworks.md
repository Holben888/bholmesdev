---
title: Understanding the spectrum of CSS frameworks
description: We've come a long way from Bootstrap. Let's explore the CSS framework spectrum from utility classes to all-in-one component powerhouses.
layout: blog-post
publishedOn: 2020-10-21T22:44:21.507Z
---

I saw a [short think piece](https://dev.to/urielbitton/stop-using-css-frameworks-3gpc) by @urielbitton recently on why people should **"stop using CSS frameworks."** It was meant to start developer discussion more than anything, since there are definitely merits to rolling your own design system from scratch. Still, with a title like that, it sparked... _quite_ the uproar in the comments 😆

I [posted my own response](https://dev.to/bholmesdev/comment/16one) on some of the nuances to explore before _writing off frameworks for good._ A kind commenter suggested it be its own post (thanks for the support y'all!), so without further ado...

_Here's my clap back_ 👏

## Understanding the spectrum of CSS frameworks

First, I'd want to draw a distinction between

1. Pre-styled, prescriptive, all-in-one frameworks like [Material UI](https://material-ui.com/) and [Bootstrap](https://getbootstrap.com/)

2. Hands-off, unopinionated, utility frameworks like [Reach UI](https://reach.tech/) and [Tailwind CSS](https://tailwindcss.com/)

I'd consider both camps to be "CSS frameworks," but for different audiences and different use cases.

## 1. The case for all-in-one frameworks

⭐️ This category is all about developer _guidance_, giving you the building blocks you need for powerful web apps (even if you don't have the domain knowledge). Above all, these are important for developers that need to focus on the business logic and product vision, but _don't_ need hyper-bespoke CSS to get there. 

### Accessible by default

The training wheels enforced by these frameworks ensure accessibility guidelines are met without as much research on the developer's part. You may not think about it, but **hacking together dropdowns and tab sliders that _look_ good won't always work well _for all people._** I recently wrote a longform post on [building an accessible navbar](https://dev.to/hack4impact/building-a-sexy-mobile-ready-navbar-in-any-web-framework-3lm2) for this very reason; there's so many little things to consider that Material UI et al. will consider ahead-of-time!

Given how mature popular CSS frameworks have become, they have a component for basically every accessibility concern you can think of. This is a huge win for teams without the resources to audit and fix a11y issues.

### Inoffensive visual identity

These sorts of frameworks also enforce a **visual identity** out of the box. Yes, this does cause websites built on Material UI, Bootstrap, etc. to feel a bit cookie-cutter. Still, these are sensible defaults to build from, _especially_ for **a)** developer teams lacking a design department and **b)** internal tools that don't need the customization effort. 

Plus, there are gray areas in frameworks like Bootstrap that _do_ allow you to build a visual identity on top of their tools. Heck, I [just worked on a project](https://www.impactlabs.io/) in this camp! We just used Bootstrap for the 12 column grid (which my team was most comfortable with using), and themed the default styles into oblivion.

That said, I do agree with a point raised by Uriel on this front:

_"Having custom styles is complicated (overriding issues) and will require multiple CSS files to be loaded."_

For full-on component libraries like Material UI, it is pretty difficult to customize beyond colors and fonts. You can quickly fall into the weeds of excessive prop passing, mismatched styling across your site, and the _infamous_ `!important` bonanza 😞

This is why I'd suggest solutions like Material UI only if your "visual identity" won't diverge from the defaults too much. If you find yourself outgrowing the walled garden though, there's always camp #2...

## 2. The case for utility frameworks

⭐️ This category is all about developer _expediency,_ providing helpful abstractions for common use cases without prescribing a look and feel to work from. These systems offer a lot more freedom by being, well, a lot smaller out of the box!

"Utility" is a very generic label that could apply to anything in UI-land. For simplicity, I'll just explore 2 major movements I've seen emerging: class-driven frameworks, and functionality-first frameworks.

### CSS class utilities - Tailwind and Bulma

Frameworks like [Tailwind](https://tailwindcss.com/) are really shaking things up. It's the first framework I've seen to go all-in on class-based styling. In other words, why can't *every* CSS property be its own class in your HTML?

At first, this sounds like a recipe for disaster. But it offers some pretty interesting benefits:

1. **Media queries are standardized right out the gate.** Instead of loosely defining breakpoints like `max-width: 860px`, you can simply append "small / medium / large" to the front of your HTML classes. For example, here's how you could dynamically change an element's size between mobile and tablet: `class="h-16 w-16 md:w-24 md:h-24"`
2. **Shorthands can speed up experienced teams,** without abstracting the CSS away too much. Since every class is a 1-1 with a CSS property (or small set of properties), you are still translating to core CSS concepts every time. This dispells the fear of using all-in-one frameworks like Material UI, where the CSS is completely hidden from developers most of the time.

This framework has a _laundry list_ of pros and cons that deserves its own article. Luckily, @swyx wrote an [amazing evaluation](https://dev.to/swyx/why-tailwind-css-2o8f) for his own needs that's worth reading 😁

I also want to mention [Bulma](https://bulma.io/) in this section, in case the breadth of Tailwind is too intimidating for you. I'd consider it a "Tailwind-lite" with a lot of the same philosophies.

### Bare-bones UI libraries - Reach UI

[Reach UI](https://reach.tech) is my favorite example in the "functionality-first" camp. In short, Reach UI gives you a set of React components with prebuilt aria labels, state management, and some lightweight CSS styles sprinkled on top (mostly for changing `display` properties). The goal here isn't to enforce a visual identity; it's to provide the _least amount of code possible_ to create usable, accessible UI elements ✨

In the end, you're free to add your own layer of styling and JS logic on top. The documentation actually guides you through overriding the defaults! I'm a CSS warrior myself, so I lean on this type of CSS framework to get things done (as long as I'm using React of course).

I'd consider this sort of framework to be **a master-class on [inversion of control](https://kentcdodds.com/blog/inversion-of-control),** and I hope more frameworks pick up on this pattern.

## Wrapping up

Overall, I think writing off CSS frameworks is a bit of a hand-wavy statement to make given the huge landscape of tools out there right now. There will always be a place for prebuilt solutions like Material UI; if anything, I see them as one step removed from powerhouses like SquareSpace or Wordpress (closer to the code, but with a lot of details abstracted away). And for teams that don't want to be tied down, solutions like Tailwind and Reach UI should fit the bill for adding handrails without losing visual identity.

Still, keep hacking away Uriel! We all have opinions, so just use what works best for you and your team 😁

## Thanks for reading! If this article was helpful...

I love writing about this sort of stuff 👨‍💻
🐦 [**Follow my Twitter for random web dev tips and articles I find cool**](https://twitter.com/bholmesdev)
📗 [**Follow my blog for new posts every 2-3 weeks**](https://dev.to/bholmesdev)