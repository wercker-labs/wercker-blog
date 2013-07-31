---
title: Adding hipchat notifications to your build and deployments
date: 2013-07-31
tags: hipchat
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
---

<h4 class="subheader">

    With the <a href="http://blog.wercker.com/2013/07/22/Announcing-the-Open-Delivery-platform.html">Open Delivery platform</a> you are now able to create your own <a href="http://devcenter.wercker.com/articles/boxes/">boxes</a> and
<a href="http://devcenter.wercker.com/articles/steps/">steps</a> for your build and
deployment <a href="http://devcenter.wercker.com/articles/introduction/pipeline.html">pipelines</a> on wercker.
</h4>

![image](http://f.cl.ly/items/0e3e432H43303U0Y450F/wercker%2Bhipchat.png)

Today, we want to share a step that we've created internally at wercker that adds Atlassian [HipChat]() notifcations to your
builds and deploys at wercker.

In order to make this happen we've also released a new phase in the [wercker pipeline](http://devcenter.wercker.com/articles/introduction/pipeline.html), after build and deploy steps, called `after-steps`, which can be used to execute steps after the build or deployment pipeline is finished. This makes them ideally suited for notifications. A followup post that deals with the `after-steps` will be released shortly on our blog!

## Adding HipChat notifications

Here is an example of an [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) for a typical Ruby application that leverages `after-steps`  for the build and deployment pipeline to send HipChat notifications.

``` yaml
box: wercker/ruby
build:
    steps:
        - bundle-install
        - script:
            name: rspec
            code: bundle exec rspec
    after-steps:
        - hipchat-notify:
            token: $HIPCHAT_TOKEN
            room_id: 628943
            from-name: wercker
deploy:
    steps:
        - heroku-deploy
    after-steps:
        - hipchat-notify:
            token: $HIPCHAT_TOKEN
            room_id: 628943
            from-name: wercker
```

The `room_id` variable is obviously the **id** for your HipChat room that you want to send the notifications to. The `from-name` is the name you want to designate to the send of the message, which defaults to **wercker**.

The `$HIPCHAT_TOKEN` can be set as an pipeline variable, which is available for the build pipeline, as well as the deployment pipeline.

![pipeline variables](http://f.cl.ly/items/0f1H0H212O0Q0d3t383R/pipeline-variables.png)

_note: this is not my real token!_

You can create HipChat tokens on the [API tokens](https://www.hipchat.com/admin/api) page.

Check out our HipChat step in the [wercker directory](https://app.wercker.com/#applications/51f26c380771b3526e000c1c/tab/details).

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.
