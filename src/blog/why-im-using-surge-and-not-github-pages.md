---
title: Why I'm using Surge and not GitHub Pages
description: "Everyone's making a static site these days, and most use GitHub Pages to get up and running fast. Here's why Surge is also worth a look 👀"
layout: blog-post
publishedOn: 2019-03-06T22:40:16.011Z
---

*Update*
Will leave a small disclaimer here that Surge hasn't seen many updates to its repo recently so be aware of that. Certainly safe to use now (haven't seen any vulnerability warnings), but worth looking out for in the future. If the project ends dying off [Netlify](https://www.netlify.com/docs/cli/) and [Zeit Now](https://zeit.co/now) are **great** alternatives that offer similar benefits over GitHub Pages 👍 

##Hm, so what's a "surge" exactly? 🤔

GitHub Pages is so prolific at this point it likely needs no introduction. [Surge](https://github.com/sintaxi/surge), on the other hand, is a much lower profile project. It accomplishes the same goal as GitHub Pages, allowing one to publish their static site on an accessible URL of whatever name they choose complete with an "https." Where surge differs is in how that website gets put on the world wide web.

Surge lets you get from 0 to published in the command line without any extra setup. Once you install the package, you can use their CLI in your project directory to whip up an account...

```
👉 surge

    Welcome to surge!
    Please login or create an account by entering email and password:

    email: jamesKPolk@napoleonofthestump.gov
    password: 
```
enter your domain to deploy to...
```
    domain: forgottenpresidents.club
```
and boom 💥! You're up and running. Note I'm using a custom URL extension here as well. You can of course specify this if you have the rights to that domain, but you can omit the extension to use the totally free `surge.sh`.

##Looks easy, but where are my git hooks?

The beauty of GitHub Pages is it will simply track your master branch and rebuild / publish your site whenever it gets updated. Thankfully, surge offers the option to hook into pushes and commits with a little addition to your `package.json`. This snippet will do the trick for redeploying on push:
```
"devDependencies": { "surge": "latest", "git-scripts": "0.2.1" },
    "git": { 
        "scripts": { "pre-push": "surge --project ./ 
        --domain forgottenpresidents.surge.sh" }
       }
```

##Neat. But what are the benefits exactly?

Surge offers a few niceties that GitHub Pages won't out of the box. For one, you obviously don't need to use GitHub to manage your version control, so you can use any custom solution you desire and still deploy with the same `surge` command.

Another is an advantage that [Zeit Now](https://zeit.co/now) also fails to address: client side routing fallbacks.

Extending our [James K. Polk](https://www.youtube.com/watch?v=StTiCU_fqCg) example, say we deployed a simple project folder with a few `html` files: 
```
    ./build
      - index.html
      - what_i_stand_for.html
      - donate_now.html
      - 200.html
```
Notice our `200.html` added at the bottom. This acts as a fallback to serve up whenever a given route is invalid. Assuming these are all the `html` files in our build, we know someone visiting, say, forgottenpresidents.club/sendfanmail would get a `404` since that file doesn't exist. However, since we have a `200.html`, Surge knows to just serve up this page instead since it's a fallback. This is more flexible than the traditional `404.html` to catch errors since we sometimes don't want certain paths to result in a `404`, like a dynamic username in the URL that gets handled in JavaScript but should always route to the same base page. This makes routing a bit cleaner whether you're using a library like React router or just tapping into the browser's history API on your own, since we don't have to deal with redirects anymore 🎉 

The last major benefit is one shared by Zeit Now: deploying whenever you want in a matter of seconds. Rather than dirtying up your commit history, you can just deploy your changes instantly and see how they look. This was super useful recently when testing out a mobile layout for a site I was developing, where mobile Safari was being a bit more finicky than my mobile viewer in Chrome. I could just deploy my changes to whatever URL I chose and see my edits as soon as I refreshed the page!

##Okay I'm starting to see it. What are the limitations?

So Surge was built to be really good at doing a very specific task. If you have anything more than a static site or client side application, there's sadly not much you can do with this tool. Any fancy backend you have would need to be deployed separately, or you can use the aforementioned Zeit Now to deploy everything in one place. Now is a much more flexible tool that's quickly gaining traction, but I've stayed loyal to Surge because of the routing niceties and lack of any necessary `config` files.

Of course you should use whatever tool that works best in your workflow. But the next time you're building v175 of your portfolio site, consider giving Surge a try 😊 

###Thanks for reading!
I'm a frontend webdev-in-training always tinkering with something. I'll try to post regularly here, so drop a follow if you enjoyed :)