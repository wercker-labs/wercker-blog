---
title: Adding email notifications to your builds and deploys
date: 2013-08-14
tags: email, notifications
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
After <a href="http://blog.wercker.com/2013/07/31/Add-hipchat-notifications.html">HipChat</a>, <a href="http://blog.wercker.com/2013/08/05/campfire-notifications-for-wercker.html">Campfire</a> and <a href="">IRC</a>notifications, you are now also able to send email notifications from your <a href="http://devcenter.wercker.com/articles/introduction/pipeline.html">build deploy pipelines</a>.
</h4>

Adding these email notifications is trivial but Bring Your Own Email Server (BYOES). We have now also [updated](http://devcenter.wercker.com/articles/werckeryml/notifications.html) our [developer center](http://devcenter.wercker.com) with all the notification documentation in place.

![image](http://f.cl.ly/items/2e1W1f0B0A0Q420E0b39/wercker%2Bemail.png)

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).

READMORE

## Updating your wercker.yml

To make use of the [Email-Notify step](https://app.wercker.com/#applications/520b5ea98a20a2624500a932/tab/details) which we've open sourced [here](https://github.com/wercker/wercker-step-email-notify), you would create a [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file in the following way:

``` yaml
box: wercker/ruby
build:
    steps:
        # Execute the bundle install step, a step provided by wercker
        - bundle-install
        # Execute a custom script step.
        - script:
            name: middleman build
            code: bundle exec middleman build --verbose
    after-steps:
        - wercker/email-notify:
            from: alerts@wercker.com
            to: mies@wercker.com
            username: $USER
            password: $PASS
            host: YOURSMTPSERVER
```

The wercker/email-notify step is self-explanatory; the **from** and **to** addresses are the email addresses from which you want to send the message and of course to whom. The **username** and **password** fields are the credentials for your SMTP server. We of course pass these along as pipeline environment variables as opposed to hardcoding them. Finally, you also fill in your SMTP **host**.

You can view the complete available options below:

#### required

* `from` - From address.
* `to` - To address.
* `host` - The host of your SMTP server.
* `username` - The username for your SMTP server.
* `password` - The password for your SMTP server.

#### optional

* `passed-subject` - Use this option to override the default passed subject.
* `failed-subject` -  Use this option to override the default failed subject.
* `passed-body` - Use this option to specify the passed body.
* `failed-body` -  Use this option to specify the failed body.
* `on` - Possible values: `always` and `failed`, default `always`


Check out our Email Notify step in the [wercker directory](https://app.wercker.com/#applications/520b5ea98a20a2624500a932/tab/details). We've open sourced the code for this step as well on [GitHub](https://github.com/wercker/wercker-step-email-notify)

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.