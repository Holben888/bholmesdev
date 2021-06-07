---
title: Grabbing subsets of JS object properties with... GraphQL?
description: It's easy to grab keys out of JSON objects, but what if we want to grab "partial" pieces of that object without destructuring?
layout: blog-post
image: assets/blog/query-json-objects-graphql/thumbnail.png
date: 2021-06-07T20:01:36.550Z
---

Messing with JavaScript objects is pretty easy these days. Destructuring syntax is convenient and the spread `...` operator helps with merging objects together. But what about grabbing just... a portion of an object?

This question deserves some visuals. Let's jump into the problem we're trying to solve, and a flexible solution we can add to any existing JavaScript project 💪

## What we want

Say I have a big document of structured data. Without manually writing a new object by hand, I just want to pull the little slices that I actually care about.

Here's one such scenario:

![A side-by-side comparison of 2 JavaScript objects. Under "what we have," there's an object with the keys "weekOldPasta" (with subways noodleSogginess, numMeatballs, and isEdible) and "panSearedSalmon" (with subkeys oilUsed, numSpices, and isEdible). On the right side under "what we want," we see the same object as before, but only using the "isEdible" subkey under weekOldPasta and panSearedSalmon. The other subkeys are grayed out.](assets/blog/query-json-objects-graphql/problem.png)

In this case, we want a copy of our original fridge, but we only care about those `isEdible` subkeys.

My gut reaction is to reach for some declarative tools in my [ES6](http://es6-features.org/#Constants) arsenal. **Object destructuring** comes to mind at first:

```js
const whatsInMyFridge = {
  weekOldPasta: {
  ...
}
const { weekOldPasta: { isEdible: pastaIsEdible },
	panSearedSalmon: { isEdible: panSearedSalmonIsEdible }
	} = whatsInMyFridge
```

There's a few issues with this:
- We can't easily destructure keys of the same name. Notice I had to convert each `isEdible` variable to the verbose `pastaIsEdible` and `panSearedSalmonIsEdible`
- Destructuring leads to some pretty gnarly code as it gets more complex. Just with a few keys, we're already hitting multi-line { curly hell }.

And most of all, **we still have to build our new object at the end!** Our destructuring statement _actually_ just made some one-off variables for each key in the object. We'll still have to do this:

```js
const whatsEdible = {
  weekOldPasta: {
    isEdible: pastaIsEdible,
  },
  panSearedSalmon: {
    isEdible: panSearedSalmonIsEdible,
  }
}
```

...which is hardly better than just writing the object from scratch 😢

What we really want is some magical syntax for _just the keys_ we want to retrieve. Something like this really:

```js
whatsInMyFridge.giveMeTheseKeys({
	weekOldPasta {
		isEdible
	},
	panSearedSalmon {
		isEdible
	}
}) // -> a beautiful formatted JS object
```

## 📈 Enter: GraphQL
If you've worked with GraphQL before, you probably noticed how close that example comes to a GraphQL query!

A brief rundown for those unfamiliar: GraphQL is a "querying" language originally built for API calls. It was mainly born from the frustrations with REST requests, as API endpoints had to _predict_ all the data a client might want to retrieve. 

[GitHub recently migrated to GraphQL](https://docs.github.com/en/graphql/guides/migrating-from-rest-to-graphql) because of this. Imagine this scenario:
- User A wants to get information about their GitHub profile. They want to send off a username and get back the accounts **name** and **profile picture**
- User B also wants some GitHub profile info. However, they're looking for a different set of info: the list of **user recovery emails** and their **personal bio**.

As you could imagine, User C might want a new combination of fields, as might users D-Z. So instead of returning a massive JSON payload to satisfy everyone, GitHub exposed a GraphQL API for you to describe _exactly_ which fields you want.

Here's how User A might request a name and profile picture as part of their request body
*This is from demo purposes, and won't actually work if you send to GitHub*
```gql
{
	userProfile(email: '10xengineer@genius.club') {
		name
		picture {
			src
			alt
		}
	}
}
```

...And GitHub will "fill in the blanks" by providing values to those requested keys. As you can imagine, this syntax is flexible enough to use on _any_ blob of JSON you want to filter down 👀

## 📖 Applying GraphQL to reading JSON objects
💡 **TLDR:** If you want the final solution without all the walkthrough, jump down to [the finished product](#link-the-finished-product)!

Let's figure out how to use that fancy syntax for our use case. The biggest question to resolve is "how do we interpret a GraphQL query in JS land?" Sadly, there isn't a nice "plain JS" solution, so we _will_ be reaching for a library here.

Go ahead and install this [graphql-query-to-json package](https://www.npmjs.com/package/graphql-query-to-json). It _does_ have a fair amount of sub-dependencies like the core [graphql package](https://www.npmjs.com/package/graphql) and the complimentary [json-to-graphql-query](https://www.npmjs.com/package/json-to-graphql-query), so if that bothers you... my apologies 😢

Let's see what we get from our old "what's edible in my fridge" request:

```js
const { graphQlQueryToJson } = require("graphql-query-to-json")
// or if you prefer: import { graphQlQueryToJson } from 'graphql-query-to-json'

const asJson = graphQlQueryToJson(`{
  weekOldPasta {
    isEdible
  }
  panSearedSalmon {
    isEdible
  }
}`)
console.log(asJson)
/* 👇
{
  query: {
    weekOldPasta: { isEdible: true },
    panSearedSalmon: { isEdible: true }
  }
}
*/
```

Neat! Toss in a string, get back a JS object. You'll notice that it wraps our requested object with the `query` key. This _would_ be useful if we were sending this request to an API, but for our purposes, we'll just ignore that key in our helper function. It also stubs out any unknown key values with `true`, which we'll use to track down unfilled values later 👀

### Traversing our query
With this JS object in hand, it's time to walk through all the keys and figure out which values to fill in. Let's start with a simple example that only goes 1 level of keys deep:

```js
const myFridge = {
	numEggs: 5,
	pintsOfIceCream: 3,
	degreeUnits: 'celsius',
}
const whatIWant = `{
	numEggs
	degreeUnits
}`
// grab the ".query" straight away, since we won't need that nested key
const whatIWantAsJson = graphQlQueryToJson(whatIWant).query
// 👉 { numEggs: true, degreeUnits: true }
```

Now we have our set of keys (`numEggs` and `degreeUnits`) each with a value of `true`. To assign our actual values in place of those `true` flags, we can
1. loop through all the object keys in `whatIWantAsJson`, and
2. assign values from the same key in `myFridge`.

```js
// loop over the object keys...
for (const key of Object.keys(whatIWantAsJson)) {
	// and if that key has a value of "true"...
    if (whatIWantAsJson[key] === true) {
		// replace "true" with the value from myFridge
		whatIWantAsJson[key] = myFridge[key]
	}
}
console.log(whatIWantAsJson)
// 👉 { numEggs: 5, degreeUnits: 'celsius' }
```

### Handling nested objects
This basic loop handles 1 level of nesting. But what if we have a request like this?
```js
{
  // level 1
  weekOldPasta {
	// level 2
    isEdible
  }
  ...
}
```

For this, we'll need a way to run our loop over `Object.keys` for _every level_ of keys in our object. Get ready to put on your computer science hat, because we're using **recursion** 😨

Pay attention to this new `else` statement we're adding:

```js
// wrap our loop in a function we can call
function assignValuesToObjectKeys(whatIWant, myFridge) {
	for (const key of Object.keys(whatIWant)) {
		if (whatIWant[key] === true) {
			whatIWant[key] = myFridge[key]
		} else {
			// if the value isn't "true", we must have found a nested object
			// so, we'll call this same function again, now starting from
			// the nested object inside whatIWant
			assignValuesToObjectKeys(whatIWant[key], myFridge[key])
		}
	}
}
```

This is a classic example of a [recursive function](https://www.freecodecamp.org/news/learn-recursion-in-javascript-by-example/). We have 2 clauses here:
- **The base case:** When we hit a value of `true`, we stop looking for nested objects
- **The recursive function call:** When we _haven't_ hit the "base" of our nested object, keep drilling down the chain of nested keys using the same function

With this in place, we have a reusable JS function for anywhere in our codebase 🥳

```js
const myFridge = {  
	weekOldPasta: {  
		noodleSogginess: “high”,  
		numMeatballs: 4,  
		isEdible: false,  
	},  
	panSearedSalmon: {  
		oilUsed: “avocado”,  
		numSpices: 3,  
		isEdible: true,  
	}
}

const whatIWant = graphQlQueryToJson(`{
  weekOldPasta {
    isEdible
  }
  panSearedSalmon {
    isEdible
  }
}`).query

assignValuesToObjectKeys(whatIWant, myFridge)
console.log(whatIWant)
/* 👉 {
	weekOldPasta: {
		isEdible: false,
	},
	panSearedSalmon: {
		isEdible: true,
	},
}
*/
```

### Cleaning this up a little
You'll notice that our `assignValuesToObjectKeys` function doesn't return anything; it just modifies the `whatIWant` object we passed in. For added readability, we might add a wrapper function to handle the `graphQlQueryToJson` call and actually `return` our requested object:

```js
function grabPartialObject(originalObject = {}, query = "") {
	const whatIWant = graphQlQueryToJson(query).query
	assignValuesToObjectKeys(whatIWant, originalObject)
	return whatIWant
}
...
const whatsEdible = grabPartialObject(myFridge, `{
  weekOldPasta {
    isEdible
  }
  panSearedSalmon {
    isEdible
  }
}`)
console.log(whatsEdible) // gives us the same object as before!
```

### Handling arrays
So we've conquered nested objects. But what if we have an _array_ of objects that we want to filter?

For instance, say our fridge data was structured just a bit differently:

```js
const myFridge = {
  food: [
    {
      name: 'Week old pasta',
      noodleSogginess: 'high',
      numMeatballs: 4,
      isEdible: false,
    },
    {
      name: 'Pan Seared Salmon',
      oilUsed: 'avocado',
      numSpices: 3,
      isEdible: true,
    },
  ],
}
```

...and we just care about the `name` and `isEdible` keys for every object in that array. Following how GraphQL requests normally work, we'd expect this sort of syntax to work:

```graphql
{
	food {
		name
		isEdible
	}
}
```

In other words, treat `food` like it's a regular object in the request, and we'll be smart enough to handle arrays of data.

This answer is a bit more involved than our previous examples. So, I'll leave you with a thoroughly commented codeblock:

```js	
function assignValuesToObjectKeys(whatIWant, myFridge) {
  for (const key of Object.keys(whatIWant)) {
    if (whatIWant[key] === true) {
      ...
	  // 👇 If the fridge data happens to be an array...
    } else if (Array.isArray(myFridge[key])) {
      // first, keep track of the object they requested
      const originalRequest = whatIWant[key]
      // then, create an array where that request used to be
      // for us to "push" new elements onto
      whatIWant[key] = []
      // loop over the items in our array of food...
      for (const fridgeItem of myFridge[key]) {
        // make a variable to store the result of assignValuesToObjectKeys
        // we use { ...originalRequest } here to create a "copy"
        const requestedItem = { ...originalRequest }
        // grab the keys we want out of that array element
        assignValuesToObjectKeys(requestedItem, fridgeItem)
        // then, push our shiny new object onto our running list
        whatIWant[key].push(requestedItem)
      }
    } else {
      ...
    }
  }
}
```

That's a fair amount of code! To briefly summarize, you'll need to:
1. Check when our actual data is an array, rather than a simple object
2. Loop over the actual data and `assignValuesToObjectKeys` for each one
3. Push the results onto a running array in `whatIWant`, with necessary helper variables to keep track of your original request

## 🚀 The finished product

Here's how our finished product looks! I've renamed `myFridge` 👉 `actualObj` and `whatIWant` 👉 `requestedObj` so our naming conventions are more universal. I also added a `hasOwnProperty` check to assert we're requesting a key that actually exists. If not, raise an exception.

Remember, you'll need to add the [graphql-query-to-json package](https://www.npmjs.com/package/graphql-query-to-json) package to your project for this to work.

```js
const { graphQlQueryToJson } = require("graphql-query-to-json")

function assignValuesToObjectKeys(requestedObj, actualObj) {
  for (const key of Object.keys(requestedObj)) {
	if (!actualObj.hasOwnProperty(key)) {
		throw `You requested a key that doesn't exist: ${key}`
	} else if (requestedObj[key] === true) {
      requestedObj[key] = actualObj[key]
    } else if (Array.isArray(actualObj[key])) {
      // keep track of the object they requested
      const originalRequest = requestedObj[key]
      // then, create an array where that request used to be
      // for us to "push" new elements onto
      requestedObj[key] = []
      for (const actualItem of actualObj[key]) {
        // make a variable to store the result of assignValuesToObjectKeys
        // we use { ...originalRequest } here to create a "copy"
        const requestedItem = { ...originalRequest }
        assignValuesToObjectKeys(requestedItem, actualItem)
        requestedObj[key].push(requestedItem)
      }
    } else {
      console.log(requestedObj[key])
      // if the value isn't "true", we must have found a nested object
      // so, we'll call this same function again, now starting from
      // the nested object inside requestedObj
      assignValuesToObjectKeys(requestedObj[key], actualObj[key])
    }
  }
}

// 👇 Function you'll actually `export` for others to use
function grabPartialObject(actualObj = {}, query = '') {
  const requestedObj = graphQlQueryToJson(query).query
  assignValuesToObjectKeys(requestedObj, actualObj)
  return requestedObj
}
```

### Usage example

```js
const { grabPartialObject } = require('./some/helper/file')

const myFridge = {  
	weekOldPasta: {  
		noodleSogginess: “high”,  
		numMeatballs: 4,  
		isEdible: false,  
	},  
	panSearedSalmon: {  
		oilUsed: “avocado”,  
		numSpices: 3,  
		isEdible: true,  
	}
}

const whatsEdible = grabPartialObject(myFridge, `{
  weekOldPasta {
    isEdible
  }
  panSearedSalmon {
    isEdible
  }
}`)
console.log(whatsEdible)
/* 👉 {
	weekOldPasta: {
		isEdible: false,
	},
	panSearedSalmon: {
		isEdible: true,
	},
}
*/
```