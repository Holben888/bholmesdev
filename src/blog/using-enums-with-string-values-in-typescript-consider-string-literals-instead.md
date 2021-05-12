---
title: Using Typescript string enums? Consider string literals!
description: Enums are a great way to standardize your types. But if you want those string values, is this the best solution?
layout: blog-post
date: 2019-07-29T14:39:03.632Z
---

If you've been using TypeScript for any amount of time, you've probably wondered this at least once:

*Can I use string values instead of numbers on TypeScript enums?*

This often comes up when you want a variable to have a select few string values. For example, say you're creating a banner for a website that's either yellow for warnings or red for emergencies. You want to make something reusable, so you add a enum for which type of banner it is:

```typescript
enum BannerType = {
    Warning = "warning",
    Danger = "danger"
}
```

This gives you a lot of flexibility of how you can use that enum's value. A common use might be defining a class name for styling your banner:

```jsx
{/* Yes, this is written a JSX-y fashion for you React users */}
<div className={BannerType.Danger}>Uh oh!</div>
```

This is much easier than writing weird helper functions and ternaries to figure out what class name to use. There are many more use cases for enum string values, like object keys, CMS content identifiers, paragraph text, error logs, etc etc etc.

## When string enums fall flat

There's a few annoyances you might find with enums + string initializers:

- They're a little verbose
- They require lookups and tooltips to see what the actual string value is
- They're limited in the special characters the enum can use

This last point was a huge point of friction for my web development team. To explain, we were looking to generate keys for content coming from the [Contentful CMS](https://www.contentful.com/). In Contentful, a key can be any string you could dream up. This means you can, say, include dots to indicate a subcategory (ex. "labels.danger") or dashes to mirror [URL slugs](https://prettylinks.com/2018/03/url-slugs/) (ex. "checkout-promo-code").

_**Clarification**: A "CMS" is an external service to host all of the content for your website. In our case, we are using Contentful to store all of the header text, body text, images, and videos we display. In order to retrieve this content, we make an API call to fetch by specific keys._

This poses a problem for our enum solution. We need to use the keys in order to retrieve the site's content, and mapping each Contentful key to an enum means tossing out all the dots and dashes! Needless to say, this could lead to some nasty collisions between keys that are unique in Contentful but not unique on our hacky enums.

## String literals to the rescue!

Luckily, TypeScript has a cleaner solution when you need those string values. You can provide a finite list of strings a variable can be assigned. Otherwise, it should throw a type error.

![Example of assigning something invalid to a string literal type](https://thepracticaldev.s3.amazonaws.com/i/vg17yj980e8yzi3s3ihj.gif)

This will also prevent you from assigning a traditional "string" type to the string literal. So, when declaring your types, you'll need to export the string literal type and use it the same way you would use an enum.

![Example of autocomplete for string literals using VS Code](https://thepracticaldev.s3.amazonaws.com/i/4l8srodbtpwfn88dp22d.gif)

You can see from the informative gif above that autocomplete works as well!

### Limitations

String literals aren't the silver bullet for every situation. Notably, using string literals doesn't improve on the verbose nature of enums. In fact, it'll often provide more information than necessary when assigning the literal type.

It's also more visually unclear what all possible values are when assigning `'random string'` instead of `SpecificTypes.Enum`. This requires team communication to decide if string literals work best for smooth PR reviewing and text editor / IDE support.
