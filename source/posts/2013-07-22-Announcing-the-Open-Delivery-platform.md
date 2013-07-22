---
title: Announcing the Open Delivery Platform
date: 2013-07-22
tags: opendelivery, boxes, steps
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
From the moment we've started wercker,
we got many requests for specific stacks; programming languages, databases and other services.

Furthermore, every developer and development team has their favorite tools, frameworks and practices.

At wercker its our mission to make developers' lives easier and we
strive to empower developers and not get in their way.

</br></br>

As such, we're very excited to announce that we're opening up our
platform so everyone can add their stack of choice to <a
href="http://wercker.com">wercker</a>.

</br></br>

In addition to opening up wercker, we now also allow you to explore your
custom stacks and steps in the wercker directory, where you can submit
and share your custom created boxes and pipeline steps in the <a
href="http://app.wercker.com/#explore">wercker
directory</a>.
</h4>

[![image](http://f.cl.ly/items/1E2V3j1p2B3p2Z0M3f2N/Screen%20Shot%202013-07-18%20at%203.17.32%20PM.png)](http://app.wercker.com/#explore)

READMORE

## The wercker pipeline

Wercker has the notion of a **pipeline**; a set of steps and phases aimed at delivering your applications.
Each time you do a git push, this pipeline gets activated.

You can read more about the wercker pipeline on our [dev
center](http://devcenter.wercker.com/articles/introduction/pipeline.html)

![image](http://f.cl.ly/items/2O3V2n3A1n2d3u3S363D/wercker_pipeline.png)

This pipeline is executed inside a sandboxed environment that we call a
**box**. Additional services, such as databases or message queues or
separate boxes, to which your application connects via [environment
variables](http://www.12factor.net/config). If all **steps** in the
pipeline are succesful, a build has passed.

## Boxes
Boxes are basically virtual machines that consist of an operating system and a set of packages installed that support your stack of choice.
For a while now wercker has had default **boxes** that support the **Node.js**, **Python** and **Ruby** programming environments.
In terms of services, we've had **MongoDB**, **MySQL**, **Postgres**, **Redis** and **RabbitMQ**.

You leverage these boxes through the
[wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file.
For instance, using the wercker default Python box is defined as follows:

``` yaml
box: wercker/python
```

Similarly for services you would use the **wercker.yml** file, as seen below for the use of the [MongoDB](http://mongodb.org) box:

``` yaml
box: wercker/python
services:
    - wercker/mongodb
```

Up until now, you were limited to using the default boxes supplied by
wercker. But starting today you can add and create your own boxes.

You can read more about boxes on our [dev
center](http://devcenter.wercker,com/articles/boxes/).

## Creating your own Boxes

Creating these boxes is trivial; you can either provision these boxes
through **Bash** based scripting or full-fledged provisioning frameworks
such as **Chef** or Puppet.

Similar to applications on wercker, boxes are defined through a single file called **wercker-box.yml**.
Boxes can be deployed to the [wercker
directory](http://app.wercker.com/#explore), which is an index of
both boxes and steps, the latter we will discuss in a bit.

Deploying your boxes to the wercker directory not only enables you to
use your own programming language or service, but lets other developers
on wercker use and improve these boxes as well!

To help you get started we've created a guide for creating a
[box](https://app.wercker.com/#explore/boxes/mies/rethinkdb) using
simple Bash scripting, loaded with
[RethinkDB](http://rethinkdb.com), an open source, distributed database,
which you can view
[here](http://devcenter.wercker.com/articles/boxes/bash.html).

Alongside
a Bash-based guide, there is also documentation for creating your boxes
using [Chef](http://www.opscode.com/chef/) that [provisions a CouchDB
box](https://app.wercker.com/#explore/boxes/mies/couchdb). You can view that guide [here](http://devcenter.wercker.com/articles/boxes/chef.html).

## Steps

Next to boxes, we've opened up the creation process for **steps**.
[Steps](http://devcenter.wercker.com/articles/steps/) make up your pipeline on wercker and define what needs to be
executed during your build and deploy process. Next to **script-based**
steps you could also use predefined steps with wercker.

Read more about [builds](http://devcenter.wercker.com/articles/introduction/builds.html)
and
[deploys](http://devcenter.wercker.com/articles/introduction/deploys.html)
on our dev center.

Examples of **buid steps** are compilation of your code, running your
unit tests or performing
[jshint](https://github.com/wercker/step-jshint/).

A **deploy step** could be the synchronization of static assets, for
which we've created the [s3sync
step](https://github.com/wercker/step-s3sync/), that takes some Amazon
Web Services
credentials and bucket information and places these assets on Amazon S3.

Steps are used in the `wercker.yml` of your application. Taking the
abovementioned steps as examples, your wercker.yml would look as
follows:

``` yaml
box: wercker/python
services:
    - wercker/mongodb
build:
  steps:
    # execute jshint
    - jshint
deploy:
  steps:
    # Execute the s3sync deploy step, a step provided by wercker
    - s3sync:
        key_id: $AWS_ACCESS_KEY_ID
        key_secret: $AWS_SECRET_ACCESS_KEY
        bucket_url: $AWS_BUCKET_URL
        source_dir: build/
```

## Creating your own Steps

Similar to boxes, steps are applications on wercker. You can create them
by including a `run.sh` file in your repository that is run on wercker
when the step is activated. Furthermore, a `wercker-step.yml` needs to
be created that contains data on the step.

We created a simple guide that creates a **Campfire Notification step**
for your builds that you can view
  [here](http://devcenter.wercker.com/articles/steps/create.html).

You can read more about steps and how to create your own on our [dev
center](http://devcenter.wercker.com/articles/steps/).

## Publishing your own boxes and steps

Boxes and steps are like any other application on wercker, a git
repository.

As boxes and steps are applications, they can be deployed. Instead of
[deploying](http://devcenter.wercker.com/articles/deployment) your application to Heroku or AWS, you
deploy (or publish) your box or step to the wercker directory.

Add your git repository containing your step or box just like any other
application on wercker and create a **wercker directory** deploy target
for it.

Upon publication, boxes and steps have an additional 'details pane' that
showcases how to use the box or step. Other metadata such as tags and
author are shown as well.

![image](http://f.cl.ly/items/2u050M0j293M2R0b0X1T/Screen%20Shot%202013-07-22%20at%203.14.32%20PM.png)

## Concluding

We're very excited about opening up our platform and can't wait to see
with what kind of boxes and steps you come up with.

By definition, steps and boxes are publically available (even if this is
a private app on wercker). We will support private boxes and steps at a
later stage.

To kickstart box development some of us created boxes for
[Golang](https://github.com/pjvds/box-golang),
[Erlang](https://github.com/mies/box-erlang),
[RethinkDB](https://github.com/mies/box-rethinkdb),
[CouchDB](https://github.com/mies/box-couchdb) and
[Meteor](https://github.com/mies/box-meteor). Apart from
the wercker boxes
available on our official [GitHub page](http://github.com/wercker).

We also have a large collection of steps again available on wercker's [GitHub
page](http://github.com/wercker), and have a bunch of steps
on hand in the [wercker directory](http://app.wercker.com/#explore).

Don't see your stack of choice or step in the wercker directory? Use
your expertise to build your own, such that others can benefit as well!

Documentation on [boxes](http://devcenter.wercker.com/articles/boxes/),
[steps](http://devcenter.wercker.com/articles/steps/) and the
[directory](http://devcenter.wercker.com/articles/directory/) are available
on the wercker [dev center](http://devcenter.wercker.com).

Explore what is currently available in the [wercker
directory](http://app.wercker.com/#explore).

## Earn some stickers!

Let us know about the boxes and steps you create for wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.
