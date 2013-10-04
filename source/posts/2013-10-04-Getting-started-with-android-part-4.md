---
title: Getting started with Android - part 4
date: 2013-10-04
tags: android
author: Jacco Flenter
gravatarhash: 7d9ef3d3f6911e6e4f9c51f6d99c48f8
---


<h4 class="subheader">
In part 4 we will do our first deploy. We've already tested <a href="/2013/09/19/Gettingstarted-with-android-part-1.html">our initial app</a> in two
different ways (see <a href="/2013/09/24/Gettingstarted-with-android-part-2.html">part 2</a> and <a href="/2013/09/27/Gettingstarted-with-android-part-3.html">part 3</a>)
</h4>

![wercker and android](/images/posts/android-part4/wanda_testflight.jpg)

This part expands the small android project created in the previous parts, so please see also:

* <a href="/2013/09/19/Gettingstarted-with-android-part-1.html">part 1</a> how to build a simple Android application using android studio version 0.2.9.
* <a href="/2013/09/24/Gettingstarted-with-android-part-2.html">part 2</a> how to test that simple application
* <a href="/2013/09/27/Gettingstarted-with-android-part-3.html">part 3</a> how to test using robolectric

READMORE

### Prerequisites

This guide expands on the code from
[part 3](/2013/09/27/Gettingstarted-with-android-part-3.html), we will not
dive too much into specific code. Going through of part 1 of the series will
also suffice.

### Introduction ###

We now have a working application that we want to deploy to services such as
<a href="https://testflightapp.com">testflight</a> or
<a href="http://hockeyapp.net/features/">hockeyapp</a> so our QA team could potentially test it.

With such services you have an overview of the devices and software your test
users use, you can distribute wercker's builds to specific groups of people
and they can get notifications when a new version is available. Both services also
have an SDK you can use, and more easily retrieve crash reports, do remote logging,
etc. In this guide we will use testflight as the primary example.


### Adding a deploy target

Since we want to deploy our application, we need to add a new deploy target to
wercker. Go to the settings tab of your application on
[wercker](https://app.wercker.com). Wercker supports a variety of deploy
targets:

* [Heroku](http://heroku.com)
* [OpenShift](http://openshift.com)
* wercker directory. This is where you can deploy your tasks/steps and boxes
to, so everybody can easily re-use them (see also [boxes and steps in the
wercker directory](/2013/07/26/Boxes-and-steps-in-the-wercker-directory.html)).
* custom deploy.

We need a **custom** deploy target. Let's call it `testflight` and save it.
![target `wercker`](/images/posts/android-part4/wercker-s2.jpg)

### Step 2: testflight api keys

Go to [testflightapp.com](https://testflightapp.com) and create an account.
Log in to the website and create a new team.

![Create team](/images/posts/android-part4/tesflight-s1.jpg)

Let's call this team 'wercker'.

![team wercker](/images/posts/android-part4/tesflight-s2.jpg)

We are presented with `Upload build` form, click on Upload API (at the bottom).

![Manual upload](/images/posts/android-part4/tesflight-s3.jpg)

Click on the 'Upload API' button.

![Upload API](/images/posts/android-part4/tesflight-s4.jpg)

Click on `get your API token` and copy the API token. Go back to wercker and edit
the custom deploy target named `testflight`. We will add the API token as an
environment variable to our deploy target. This way, we can specify how to deploy
inside the [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) and use that for different deploy targets (for instance
staging and production).

So click on `+ add new variable` in the `Deploy pipeline` section and create
an environment variable. Call it `API_KEY` and make it protected. This way
it will by default not be visible for instance in the setup environment
variables step and will not be send again to the frontend.

![target `wercker`](/images/posts/android-part4/wercker-s3.jpg)

On the testflightapp website, let's go back to the `Upload API` page and `Get your
team token`. Now add another environment variable named `TEAM_TOKEN`

`Save` it and we're ready for the next step.

### Updating the wercker.yml

In the previous parts we've always added steps to the build pipeline in the
wercker.yml. This time we want add steps to the deploy pipeline. So let's add the following snippet
at the bottom of the `wercker.yml`:

``` yaml
deploy:
  steps:
    - script:
        name: upload to testflight
        code: |
          curl http://testflightapp.com/api/builds.json -F file=@GettingStarted/build/apk/GettingStarted-debug-unaligned.apk -F api_token="$API_TOKEN" -F team_token="$TEAM_TOKEN" -F notes="Deploy of commit: $WERCKER_GIT_COMMIT from branch: $WERCKER_GIT_BRANCH" -F notify=False -F distribution_lists=''
```

We use steps, similar to our build pipeline and all it is, is a basic curl
command. It is important however to keep this as a single line command.
Multiline commands can sometimes fail on wercker.

What are the parameters for the api call:

- api_token
- team_token
- notes. These are the release notes.
- notify. Whether users should be notified or not.
- distribution_list. A list of groups that can access the application

Time to test our deploy:

```
$ git commit -am "deploy steps added"
$ git push
```

When the build is finished we can trigger the deploy from the build page on
wercker. We can deploy green builds by clicking the `deploy this build`
dropdown and selecting `testflight` from the list.

The browser will be redirected to the deploy page, showing the progress of the
deploy.

Wait until the deploy is completed and expand the `upload to testflight` if
all went right you should see a json response object.

Switching back to testflight, we should see in the **apps list** our GettingStarted
app. Click on applications and go to 'wercker' via the breadcrumbs. Let's
enable notifications when a new build is deployed. For this, click on 'wercker'
and add a distribution list, named 'wercker' and add at least one user.

We also need to update `wercker.yml` to set the `notify` option to True and
set the distribution_lists to 'wercker'. Commit/push the changes and deploy the
build. The user should receive an email!

You now have continuous delivery set up for your Android applications!


---

Have fun!

### Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).
