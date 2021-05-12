---
title: A neat DIY solution to animating on scroll (for any framework)
image: https://thepracticaldev.s3.amazonaws.com/i/obp112knr1cvmpa0kjlu.png
description: AOS libraries sure are slick, but why add dependencies when it an be simple to implement yourself?
layout: blog-post
date: 2019-04-09T15:46:47.093Z
---


> **Update 4.10.19** A recent comment below blew my mind name dropping the [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API). This was essentially built for exactly what this article's about, allowing you to fire a callback whenever an element is in a certain position on the screen! The latter parts of the article about centralizing logic with a store are still totally applicable to this, but intersection observers clean up things a great deal over `requestAnimationFrame` while also being more performant. However, note there is no IE support and Safari support only just recently. Alrighty, enjoy the rest of this post 😊

Looking around the world wide web for inspiration, I've found that many sites I love incorporate fun little "reveal" animations whenever I scroll to certain elements. Though subtle, these extra touches make the page feel much less static and more **responsive**. The question is though... what's the best way to implement this?

Just scrolling through CodePen examples, I have found [time](https://codepen.io/syedrafeeq/pen/yEJKn) and [time again](https://codepen.io/letsbleachthis/pen/zewKYE) that people are reaching for catch-all libraries that can handle it for them. There are countless options out there for animating on scroll, the most prevalent being the aptly named [AOS](https://github.com/michalsnik/aos). I myself was hoping to 🌶 up my site with some scroll animations, so I naturally thought to turn to AOS library for this. However, as my implementation grew more and more specialized (ex. how to I avoid loading this iFrame until I scroll to it?) I began to wonder...

*Can't I just build this myself?*

## Maybe. Let's see how

Just starting with basic, vanilla JS and no frameworks, the approach is actually pretty simple. All we need is an `onScroll` handler and whatever elements we actually want to animate. Starting with the basics, say we have an element of a specific ID we want to trigger an animation for. As you might imagine, we can reach for the DOM window's `onScroll` event to figure out where our element is on screen whenever you, well, scroll:

```javascript
window.onScroll = ({target}) => {
    const element = document.getElementById('animate-me')
    const elementTop = element.getBoundingClientRect().top
    if (elementTop < document.body.clientHeight) {
        element.classList.add('scrolled-to')
    }
}
```

There are a few nested object attributes we need to grab for this. First, we need to get the pixel value for where the top of the element is on screen. There are a few valid ways of finding this, but through a quick internet search it seems `getBoundingClientRect()` is the most reliable way of doing so across browsers. 

With this, we should compare against the fixed height of the document. This is basically just the height of your browser window, being the `clientHeight`. If the top of our element is less than this height, then some part of it must be on screen. Now, we just add our keyframe to our CSS on `.animate-me.scrolled-to` and we're good to go 👍 

### Okay great, we basically recreated a MDN help page example...

With that out of the way, let's actually make this useable in the real world. Firstly, if you got curious and threw a `console.log` statement in there, you likely got this whenever you twitched your scroll wheel.

![Console log of on scroll handler](https://thepracticaldev.s3.amazonaws.com/i/defuaddw82tdkyrzkfpo.png)

This reflects how expensive analyzing every scroll event actually is. We're executing a function for every pixel we scroll, and as we start making this function more robust, that can start causing lags and stutters. 

One way to resolve this is using a `requestAnimationFrame` to decide when our callback gets fired. This is another window-level function where you may queue up callbacks for the browser to call. When it feels it is ready to execute those functions without un-buttery-smoothing your scrolling experience, it will fire them off. Thankfully, this approach has seen [relatively high browser adoption](https://caniuse.com/#search=requestanimationframe). All we need is a wrapper around our `onScroll` handler to `requestAnimationFrame`, along with a `boolean` flag to let us know whether or not our previous callback is done executing:

```javascript
let waitingOnAnimRequest = false

const animChecker = (target) => {
    // Our old handler
    const element = document.getElementById('animate-me')
    const elementTop = element.getBoundingClientRect().top
    if (elementTop < document.body.clientHeight) {
        element.classList.add('scrolled-to')
    }
}

window.onScroll = ({target}) => {
    if (!waitingOnAnimRequest) {
        window.requestAnimationFrame(() => {
            animChecker(target)
            waitingOnAnimRequest = false
        })
        waitingOnAnimRequest = true
    }
}
```

Great! Now our calls should be a bit more efficient. But let's address a more pressing issue: how do we get this working for **any** element in the document we may want to animate on scroll?

It certainly wouldn't make sense to keep adding callbacks for each possible ID or className we would need, so why not just create a centralized array we can append all of our element selectors to?

## Time for some loops

This addition is fairly straightforward leveraging `querySelectorAll`. Just create a global array with all selectors that should animate (either IDs or classes) and loop over them like so:

```javascript
let animationSelectors = ['#ID-to-animate', '.class-to-animate']

const animChecker = (target) => {
    // Loop over our selectors
    animationSelectors.forEach(selector => {
        // Loop over all matching DOM elements for that selector
        target.querySelectorAll(selector).forEach(element => {
            const elementTop = element.getBoundingClientRect().top
            if (elementTop < bodyHeight) {
                 element.classList.add('scrolled-to')
            }
        })
    })
}
...
```

Now our scroll animation checker should be able to handle any element we throw at it!

## Neat! But I use X framework...

...and I don't think I could use this because of Y 🤷‍♀️

Now hold it right there. I understand everyone's tooling has its own set of quirks, so let's try to address some of them.

### I use a component system, so how do I centralize this logic?

Though it would be nice to have a succinct list of classes and IDs we would want to animate, components, especially with scoped CSS solutions, make it difficult to keep this list readable and expandable.

Thankfully, this solution just needs a single array of strings to get working, so we can use a global store each component can update with the DOM selectors they want to animate. I used this in a recent project  built on SvelteJS, which uses a subscription-based global store. To update `animationSelectors`, I just created it as a store...

```javascript
export const animationTriggers = writable({})
```

... and added the class name from whichever component when it gets created.

```javascript
import { animationTriggers } from '../stores'

onMount(() => {
    animationTriggers.set([
      ...$animationTriggers,
      '.wackily-animated-class',
      '#section-id',
    ])
  })
```

This works just as well for common global state solutions like Redux and React Context as well. Redux implementations widely vary by middleware so I'll spare the multi-file example here, but here's an option using React Context (which works in vanilla React):

```javascript
// store.js
...
const AnimationTriggerContext = React.createContext()

class StoreWrapper extends React.Component {
    constructor() {
        super()
        this.state = {
            selectors: []
        }
    }
    render() {
        return (
            // create a provider to wrap our components in at the parent level
            <AnimationTriggerContext.Provider value={{
                // make our array of selectors accessible from all children
                selectors: this.state.selectors,
                // add a helper function to update our array
                addSelector: (selector) => {
                    this.setState({
                        selectors: [...this.state.selectors, selector],
                    })
                }
            }}>
                {this.props.children}
            </AnimationTriggerContext.Provider>
        )
    }
}

//childManyLayersDeep.js
...
class Child extends React.Component {
    componentDidMount() {
        this.context.addSelector('special-class')
    }
    render() {
        return <div className="special-class"></div>
    }
}

//wrap the child with a 'withContext' so it can be accessed
export default withContext(Child)
```

Naturally, this method is extendable to VueJS, RxJS observables, and basically everywhere else you might use a global store.

### Okay that's pretty nifty... but I can't use basic CSS selectors. These are components!

Okay fair point; this can complicate things in most component-based frameworks. The simplest compromise is to pass a reference to the element itself in our "add" function instead of the class name so we can avoid DOM querying. Overall, the humble `ref` attribute in React or Vue, rather than a class or an ID selector, should do the trick for this.

### Also, I'm using CSS-in-JS and would rather not check for class names to start animating. What are my options?

This is a fairly common pattern these days and tends to rely more on prop passing that on class name switching. Thankfully, we have pretty much all of the logic in place to figure out these props based on our store. All we need is an extra object attribute on the selector we pass in, say a `scrolledTo` flag, which can be set "true" or "false". 

For this, we would modify what we add to our store to go from just a string (or ref) to an object...

```javascript
{
    selector: 'class-name',
    scrolledTo: false,
}
```
...and update its flag when scrolled to.

```javascript
const animChecker = (target) => {
    ...
        if (elementTop < bodyHeight) {
            animationTriggers[currentIndex].scrolledTo = true
        }
    ...
}
```

Now we can subscribe to our array of animationTriggers (or grab the context, depending on your implementation) and pass our `scrolledTo` flag as a prop to the component's styles.

## In Summary

So before you protest that you could have gotten your favorite animate-on-scroll library working in the time it took to read this article... I get it. But I would say taking this feature as a fun little challenge to build out yourself is super helpful for understanding how to make sleek, efficient DOM listeners. It also means you will have one less dependency to worry about in your package, so no breaking changes and a lot of flexibility for adding new features!

To see this solution in action, it is used all over the place on our Georgia Tech club's homepage: Golden Swarm Games. [Visit the site](https://gsg.surge.sh) or [the repo](https://github.com/Holben888/gsg-site) to see how our scroll animations work under the hood. 
