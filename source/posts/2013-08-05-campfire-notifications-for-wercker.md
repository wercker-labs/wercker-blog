---
title: Adding Campfire notifications to your builds and deploys on wercker
date: 2013-08-05
tags: campfire, notifications
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
<a href="http://blog.wercker.com/2013/07/31/Add-hipchat-notifications.html">Last week</a> with HipChat notifications, we introduced the 'after-steps' functionality on wercker, which allows you to create hooks to be called after builds or deploys.
    With the <a href="http://blog.wercker.com/2013/07/22/Announcing-the-Open-Delivery-platform.html">Open Delivery platform</a> you are now able to create your own <a href="http://devcenter.wercker.com/articles/boxes/">boxes</a> and
<a href="http://devcenter.wercker.com/articles/steps/">steps</a> for your build and
deployment <a href="http://devcenter.wercker.com/articles/introduction/pipeline.html">pipelines</a> on wercker. In this post we want to showcase how to add <a href="http://campfirenow.com">Campfire</a> notifications throught the 'Campfire-Notify-Step'
</h4>

![image](http://f.cl.ly/items/263n40133k442U3E1p2A/wercker%2Bcampfire.png)

The `after-steps` phase in the [wercker pipeline](http://devcenter.wercker.com/articles/introduction/pipeline.html) allows you execute steps after the build or deployment pipeline is finished, which is perfect for notifications. We created the [Campfire-Notify step](https://app.wercker.com/#applications/51f2a3e8df5a46247c000e0d/tab/details) internally at wercker to shoot off messages on failed deploys.

## Adding Campfire notifications

Here is an example of an [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) for a typical Ruby application that leverages `after-steps`  for the build pipeline and sends notifications to a Campfire room after builds.

``` yaml
box: wercker/ruby
build:
    steps:
        - bundle-install
        - script:
            name: rspec
            code: bundle exec rspec
	after-steps:
        - campfire-notify:
            token: $CAMPFIRE_TOKEN
            room-id: id
            subdomain: campfiresubdomain
```

The `room-id` variable is obviously the **id** for your Campfire room that you want to send the notifications to.

The `$CAMPFIRE_TOKEN` can be set as an pipeline variable, which is available for the build pipeline, as well as the deployment pipeline.

![image](http://f.cl.ly/items/1R0L3O0c0m3t2p1N0F2M/Screen%20Shot%202013-08-05%20at%201.20.40%20PM.png)

_note: this is not my real token!_

Check your info/account page for your API token that you can use as the above mentioned environment variable.

View our Campfire step in the [wercker directory](https://app.wercker.com/#applications/51f2a3e8df5a46247c000e0d/tab/details).

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.
