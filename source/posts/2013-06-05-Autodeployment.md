---
title: Autodeployment with wercker
date: 2013-06-05
tags: autodeployment, feature, wercker
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
Auto-deploying your application with wercker
</h4>

This week we've added a new feature to wercker that streamlines your deployment process even more.

You are now able to auto-deploy your build for specific [deploy targets](http://devcenter.wercker.com/articles/introduction/deployment.html) (dev, staging, production).

However, [with great power comes great responsibility](http://en.wikiquote.org/wiki/Stan_Lee). Internally we use auto deploy for our [dev center](http://devcenter.wercker.com) but not for mission critical services. We can imagine developers leveraging auto-deploy for staging environments, feature branch deploy targets or, for instance, their blogs.

In a previous [post](http://blog.wercker.com/2013/05/31/simplify-you-jekyll-publishing-process-with-wercker.html) we've outlined how to build and deploy your Jekyll-based blog to S3 with wercker; an ideal candidate for auto-deployment.

![image](http://f.cl.ly/items/2R1a1Y3V0r3k2A2j3U0P/Screen%20Shot%202013-06-03%20at%203.18.49%20PM.png)

You are also able to specify which [git branch](http://git-scm.com/book/en/Git-Branching-Basic-Branching-and-Merging) you want auto deployed, ideal for when you are working on a new feature.

Let us know what you think!