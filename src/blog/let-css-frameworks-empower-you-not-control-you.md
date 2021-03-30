---
title: Let CSS frameworks empower you, not control you
image: https://thepracticaldev.s3.amazonaws.com/i/qpp0oqach2mmmq300eqp.png
description: CSS frameworks are super useful for the building blocks of a website, but when is it best to start adding custom styles?
layout: blog-post
publishedOn: 2019-04-29T14:54:27.803Z
---

Many articles tackling the issue of CSS frameworks either argue [whether or not they should be used in a project at all](https://dev.to/teamxenox/do-we-really-need-a-css-framework-4ma6), or dig deep on the pitfalls of each potential option. From my experience, there's a bit more nuance to it than that. CSS frameworks have a clear place to take care of all the hefty CSS logic that grows hard to maintain, like accessibility concerns, modal / pop-up visiblity, desktop vs. mobile nav bars, and so on. 

But where do you draw the line on what should be framework-infused and what should be custom? More to the point, how do you stop feeling so controlled by the framework that nothing you make has that special touch?

## My philosophy

I had to encounter this issue recently when joining a project team in the Bits of Good organization at my university, Georgia Tech (go yellow jackets! 🐝). I inherited a project a semester in progress that needed some serious wind in its sails to hit the finish line, but there was one little hitch: the site was thoroughly entrenched in [Bootstrap](https://getbootstrap.com). 

I personally **love** whipping up small design systems for new projects, so switching to a CSS framework felt like a bit of a chore at first. However, I found some clear utility in it: it was now much easier to, say, include popover modals with background fade, or use decent buttons and dropdowns without worrying about tab focus. With this, I found the best way to reconcile my world of customize-all-the-things with my team's lego-bricking approach was to **treat frameworks as a utility.**

CSS frameworks shouldn't be the driving force behind how elements  look and feel, nor should they govern the flexibility of the site at different screen sizes. Rather, they should be a place to grab those little bits and pieces when custom solutions stop making sense. I understand that this philosophy is hard for some to justify when they need that homepage done by yesterday; still, I think this can work at any level with a little adjustment of where the custom stops and where the framework utility begins. With that, here are the main points I want to expand on.

## Don't rely on cookie-cutter layouts.

This is a pretty major problem I've seen, as fairly simple layouts get overly complicated by the constraints of framework layout styling. Though recent efforts like [Semantic UI](https://semantic-ui.com/examples/responsive.html) have done a better job of adding flexibility to the fold, there are still some severe rough edges that can make writing markup take longer than necessary.

I saw an example of this recently when a teammate was working on a pretty straightforward screen for our app: an applicant info view. This was just meant to be a list of headers and paragraphs to describe someone's name, email, address, personal bio, and so on. However, the curse of Bootstrap caused the document to become... this.

```html
<Container>
  <Row>
    <Col>
      <Heading>{`${applicant.bio.first_name} ${applicant.bio.last_name}`}</Heading>
    </Col>
  </Row>
  <Row>
    <Col>
      <h5>
        <Label for="email">Email</Label>
      </h5>
      <p className="content">{applicant.bio.email}</p>
    </Col>
    <Col>
      <h5>
        <Label for="phoneNumber">Phone Number</Label>
      </h5>
      <p className="content">{applicant.bio.phone_number}</p>
    </Col>
    <Col>
      <h5>
        <Label for="birthDate">Birth Date</Label>
      </h5>
      <p className="content">{applicant.bio.date_of_birth}</p>
    </Col>
  </Row>
... a lot more row / columns
```

This is what I refer to as "spreadsheet hell," where seemingly everything on the page feels like it has to be a row or column in some grand table defining the site. Thankfully, this is using the popular component framework React, so I can spare you the more laborous `<div class="col xs-4-72-57-sqrt(pi)">` on every tag 😉

Yes, crafting containers from scratch can be tough. But once you start using [Flexbox](https://medium.freecodecamp.org/an-animated-guide-to-flexbox-d280cf6afc35) or [CSS grid](https://cssgridgarden.com) more often, getting your HTML chunks to appear where you want can start to feel much quicker just writing the CSS directly. 

In sum, to reduce overhead and make layouts work well, take a stab at some custom styling before letting the framework do it for you.

## Tweak that border radius if it feels right

Another issue with CSS frameworks is how they encourage "window shopping," where you start to design your website around the default components. This works for getting up and running fast, but I'm sure you've visited countless sites that feel a bit too... bootstrap-y.

This is a more difficult hurdle that can be framework-specific. Most offer theming options for color and font overrides, but decidedly little beyond that. So, I find it helps to start picking which components of the site can have a more unique feel, and which can stay in framework-land. This is a fairly arbitrary choice I say is best left to your design preferences and what you can build with your CSS skills. However, changing something small like adding a textured background or a call-to-action button with a dramatic drop-shadow can do a lot to make a site feel like your own.

Here's a simple example from our own site. We had just added content search functionality to the applicant viewer, wiring up all the frontend and backend logic to get it working how we wanted. However, presenting it to our client, we found an issue: it was unclear how to change what you are searching by. This is because we were using a default bootstrap button for the dropdown icon, so it wasn't clear how it was associated with the searchbox. So, we broke out some custom CSS to change the button's background color and push it closer to the input box, complete with a unifying border radius. The end result felt much cleaner!

![search dropdown with snazzy styling](https://thepracticaldev.s3.amazonaws.com/i/w5cu0izo08fgnf4jmumn.gif)

## Don't let frameworks decide your functionality

This is the biggest point to drive home, since it's rare that a CSS framework will cover **every** possible element you can have on your site.

Our team ran into this pretty early on when building out our administrator dashboard. In short, we were designing an applicant viewer where an admin could quickly switch between volunteer applications to view applicant info and change their approval status. After some discussion, we decided that something similar to an email panel might work best, with an overview of applicants and their statuses along the left, and a space for each full application on the right.

We certainly could have gone window-shopping through the sea of bootstrap components to find inspiration. However, this probably would have steered us towards some sub-optional pull-out menu or tab view that didn't quite hit the mark. This goes back to my point that frameworks are really a utility at the end of the day; if they can't help you out, don't reach for them!

![Our final admin dashboard, complete with charmingly repetitive dummy data](https://thepracticaldev.s3.amazonaws.com/i/gxb9hn7cxyuz980muvqv.png)

_Here's our final administrator dashboard, complete with charmingly repetitive dummy data!_

## To sum up

Building websites is hard. That shouldn't surprise anyone. It can be easy to see CSS frameworks as a catch-all to build a site from scratch with no experience required, but they all come with their own sets of gotchas and limiations that can hamper the quality of your site. When working on a new page, I'd say just consider the following:
1. What do you want this to do?
2. What navigation, layout, buttons, etc. could best help you do that?
3. Is there any overlap between that and the CSS framework you are using?
4. If so, does the styling best suit your needs, or could it use a 🌶 switch-up?

## Thanks for reading! 😊
I'm a frontend webdev-in-training always tinkering with something. I'll try to post regularly here, so drop a follow if you enjoyed!