---
title: Spotlight on wercker's design process
date: 2013-07-15
tags: design, internals
author: Lindsey Bateman
gravatarhash: e1c82876f21cdafafd2b01a1e625f587
---

<h4 class="subheader">
Looking back at wercker six months or so, it's incredible to see the progress we've made, not only from a functional standpoint but also from a product design perspective.
In this post, we briefly want to discuss our internal design process and the tools we use for designing wercker.
</h4>

![image](http://f.cl.ly/items/1D1q0l0P1r1Q0O0g131L/design_post_1.png)

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).

READMORE

At wercker we want to move forward with great velocity so our sprints consists of 5 days including UX and design. This also allows for a short feedback cycle with our customers, improving our product at a rapid pace, release new features and gather feedback. Rinse and repeat.

## Sketch

We translate designs on whiteboards or paper into mockups quickly using [Sketch](http://www.bohemiancoding.com/sketch/). Sketch is a professional vector graphics tools with a minimal interface which is easy to use. It's ideal for designing interfaces and quick UX mockups. It took some time getting used to coming from Photoshop, but Sketch is extremely lightweight and a joy to use.

![image](http://f.cl.ly/items/2e271R2C3T0P1a212b1s/design_post_2.png)

If our mockups look good in terms of ux and design, we immediately proceed to implement the design in HTML. If the feature or mockup is too large with regards to scope, we cut it up into smaller increments.
We previously wrote about our [new add application flow](http://blog.wercker.com/2013/06/21/Introducing-our-new-add-app-flow.html), which we designed and implemented in this fashion; we explored various directions in Sketch and subsequently settled on one specific design that we implemented.

## Design in HTML

At wercker we've created a styleguide for all our UI elements and leverage a grid system. As such, we're able to quickly translate Sketch-based wireframes to our frontend. Through this process we don't spend copious amount of time in Photoshop and immediately proceed with HTML and CSS, for which I'm also responsible, again allowing for quick iterations. There is no going back and forth between tech and design, as they're the same person. This also means that we're directly able to see what the impact of live data is with the implemented design. A design is usually based on the most advantageous use-case, but actual data quickly knows how to change this with a vengeanc ;-)


## Responsive

Are you responsible for testing your applications source code? Do you deploy? Or a do you just want to keep tabs on builds and deploys remotely?
At wercker we think its important that you are able to use our product on multiple devices. Mobile phones are perfect for casually checking the status of your project on wercker. As such, the wercker platform has a mobile first design and is completely responsive.

In order to realize this we picked [Foundation](http://foundation.zurb.com) from Zurb. Foundation works with [Sass](http://sass-lang.com/), which is easy to style and comes with several components that are ready to use.

It also enables us to shift pieces around on Foundation's grid. It obviously comes with excellent handles that account for different devices and media queries.

## Icon font pack

![iamge](http://f.cl.ly/items/0d083c1m3d1A3E0k243S/design_post_3.png)

We wanted a font for all icons we've created and will create in the future; vector icons which are to be placed inside buttons, behave as text, can be colored and remain razor sharp when enlarged.
For this we use the [Inkscape](http://inkscape.org/) and [FontForge](http://fontforge.org/) combination allowing us to quickly create and add an icon to our set.

We hope this post has been helpful for your own design purposes and let us know if you have any feedback. Follow this blog for future updates on wercker and our design process!

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Thanks for tuning in, and there will be more posts next week! Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.