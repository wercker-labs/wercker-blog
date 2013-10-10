---
title: Extending wercker with an external service
date: 2013-10-10
tags: wercker steps heroku wercker
author: Jacco Flenter
gravatarhash: 7d9ef3d3f6911e6e4f9c51f6d99c48f8
---


<h4 class="subheader">
    wercker is an open platform and can be extended and enhanced by utilizing
    easily reusable steps and boxes.
</h4>

You can find them in the <a href="https://app.wercker.com/#explore">wercker directory</a>.
What if you can't find what you are looking for? Create your own. So let's see how to create a step.

READMORE

#### Introduction

While creating [the](/2013/09/19/Gettingstarted-with-android-part-1.html)
[series](/2013/09/24/Gettingstarted-with-android-part-2.html)
[on](/2013/09/27/Gettingstarted-with-android-part-3.html)
[android](/2013/10/04/Getting-started-with-android-part-4.html) development I
found that I wanted wercker to set the version numbering for me. Each release
should out increment some counter. The solution we will explore here will use
an external website (running on a free heroku instance) which can supply us
with a build number.

#### The basics

What is a step? We envision steps as basic blocks of functionality which are
contained and might be reused for different applications. So instead of using
the script step for everything, you can create a step for actions that are
a bit more complex or that may use certain options/switches you otherwise have
to copy/paste from that other project. Typically steps do one of the following:

* notify users (for instance via
[email](https://app.wercker.com/#applications/520c938f8a20a2624501003e/tab/details),
[irc](https://app.wercker.com/#applications/5235dd91dfe78bb24c001b1f/tab/details),
[campfire](https://app.wercker.com/#applications/51f2a3e8df5a46247c000e0d/tab/details)
[flowdock](https://app.wercker.com/#applications/5232a884341102d86800611b/tab/details)).
* install software. Example:
the [bundle-install](https://app.wercker.com/#applications/51c829d13179be44780020be/tab/details)
step
* run tests, validate code or run some kind of lint. Like the
[validate-wercker-step](https://app.wercker.com/#applications/51cd593c7578aa5b5300026b/tab/details)
* generate an environment variable or store some data in a certain location.
The [add-ssh-key](https://app.wercker.com/#applications/523afff01aa016c8590015b1/tab/details) step
does this

Our step will create an environment variable that we can use later during the
build and update whichever file we need to for our specific language.

### The pipeline

Whenever you start a build or a deploy there are a couple of basic steps that
are set in stone in the wercker pipeline:

* get code. During builds this is a git clone, during deploy the build output
is retrieved.
* setup environment. During this step the boxes are started. Based on the
wercker.yml
* saving build output (builds only)

In our wercker.yml, there are two opportunities in the pipeline to insert our
own logic, which are defined in the wercker yaml as:
* 'steps'. This is run right after setup environment
* 'after-steps'. They are run after a build/deploy is finished (succesfully or
not).

The steps are executed in order and if one fails, the next ones after are not
executed. The after-steps are different, they are basically run outside of the
build flow and are meant for steps that need to be run, no matter the outcome
of the build or deploy (or specifically when a build/deploy is failed).
This is particularly useful for sending notifications.

### A steps' lifecycle

When wercker runs a step a couple of environment variables. Let's take a look
at them:

<table border="0">
<tr>
    <td>WERCKER_STEP_ROOT</td>
    <td>/wercker/steps/wercker/bundle-install/0.9.1</td>
    <td>The path to the working directory of the step that is currently executed. It contains the full content as deployed to the <a href="http://app.wercker.com/#explore">wercker directory</a>
</tr>
<tr>
    <td>WERCKER_STEP_ID</td>
    <td>9c182f44-e12d-4daf-91eb-a48d0540cc10</td>
    <td>The unique - within the context of the pipeline execution - idenfier for the step. The pattern is _{STEPNAME}{ORDINAL}. The value could be different on the next run of the pipeline
</tr>
<tr>
    <td>WERCKER_STEP_NAME</td>
    <td>S3SYNC</td>
    <td>The name of the step as specified by the step in <strong>wercker-step.yml</strong>
</tr>
</table>
