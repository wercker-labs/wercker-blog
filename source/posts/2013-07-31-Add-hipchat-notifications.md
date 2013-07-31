---
title: Adding hipchat notifications to your build and deployments
date: 2013-07-31
tags: hipchat
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
published: false
---

Today we are happy to announce hipchat notification support for your build and deployments at wercker. It is released in the form of a step and a new pipeline phase called `after-steps`, which can be used to execute steps after the build or deployment pipeline is finished. This makes them perfect for notifications.

## Adding HipChat notifications

Here is an example of an `wercker.yml` for a typical ruby application that leverages the `after-step` in for the build pipeline as well as the deployment pipeline to send hipchat notification.

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

The `$HIPCHAT_TOKEN` can be set as an pipeline variable, which is available for the build pipeline, as well as the deployment pipeline.

![pipeline variables](images/posts/add-hipchat-notifications/pipeline-variables.png)

You can create HipChat tokens on the [API tokens](https://www.hipchat.com/admin/api) page.
