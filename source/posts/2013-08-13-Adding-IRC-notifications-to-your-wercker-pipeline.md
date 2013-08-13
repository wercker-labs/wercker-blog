---
title: Adding IRC notifications to your builds and deploys
date: 2013-08-13
tags: golang, boxes
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
After <a href="http://blog.wercker.com/2013/07/31/Add-hipchat-notifications.html">HipChat</a> and <a href="http://blog.wercker.com/2013/08/05/campfire-notifications-for-wercker.html">Campfire</a> notifications, we're now happy to publish a post on how to add IRC notifcations to your build and <a href="http://devcenter.wercker.com/articles/introduction/pipeline.html">deploy pipelines</a>.
</h4>

![image](http://f.cl.ly/items/0i1O3D0e3a1U2R150I3Z/wercker%2Birc.png)

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).

READMORE

## Updating your wercker.yml

IRC notifications also make use of the `after-step` functionality allowing you to execute steps after the build or deployment pipeline is finished which is ideally suited for notifications.
We created the [IRC-Notify step](https://app.wercker.com/#applications/51f2a14ddf5a46247c000cf7/tab/details) internally at wercker to shoot off messages on passed or failed builds for our projects. You can check out the source code for this step [on GitHub](https://github.com/wwwouter/wercker-step-irc-notify).

You make use of the **IRC Notify step** by updating your [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/), which is the way to compose your build and deploy pipeline on wercker. Below is an example to add notifications on either failed or passed builds for this blog:

```yaml
box: wercker/ruby
build:
    steps:
        - bundle-install
        - script:
            name: middleman build
            code: bundle exec middleman build
    # use IRC NOTIFY Step
    after-steps:
     - wouter/irc-notify:
        server: irc.freenode.net
        port: 6667
        nickname: wercker
        channel: wercker-dev
```

As you can see in the `after-steps` clause, we make use of the `wouter/irc-notify` step, as [wouter](https://app.wercker.com/#wouter) created this notifier. We fill in the server we wish to connect to, which port and under which **nickname** we want to publish the message. Finally, we declare to which irc **#channel** you want to post the message to. You can view all the options available below:

#### required

* `server` - The hostname or ip address of the IRC server.
* `port` - The port of the IRC server.
* `nickname` - The nickname of the user that sends the message.
* `channel` - The channel that the message will be send to. Do not include '#' sign.

#### optional

* `passed-message` - Use this option to override the default passed message.
* `failed-message` -  Use this option to override the default failed message.
* `on` - Possible values: `always` and `failed`, default `always`

And now you have IRC notifications on your builds and deploys, as can be seen below:

![image](http://f.cl.ly/items/2e3y2M2E1r2Q0g3M1u44/Screen%20Shot%202013-08-13%20at%206.49.38%20PM.png)

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.
