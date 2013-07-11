---
title: Announcing the Open Delivery Platform
date: 2013-07-10
tags: opendelivery, boxes, steps
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
published: false
---

<h4 class="subheader">
From the moment we've started wercker, we got many requests for specific stacks; programming languages, databases and other services. As a startup, our time and resources however are limited and as such we are unable to cater towards every need. More importantly, we've felt that we should not be the one to dictate your programming needs; every developer and development team has their favorite tools, frameworks and practices. As a platform, it is our mission to empower developers and not get in their way. As such, we're opening up our platform so everyone can add their stack of choice to <a href="http://wercker.com">wercker</a>.
</h4>

![image](http://f.cl.ly/items/412f0n0f1B0E1Q1m3o0L/wercker_home_screenshot2.jpg)

READMORE

## The wercker pipeline

Wercker has the notion of a **pipeline**; a set of steps and phases aimed at delivering your applications.
Each time you do a git push, this pipeline gets activated.

![image](http://f.cl.ly/items/2O3V2n3A1n2d3u3S363D/wercker_pipeline.png)

This pipeline is executed inside a sandboxed environment that we call a **box**. Additional services, such as databases or message queues or separate boxes, to which your application connects via [environment variables](http://www.12factor.net/config)/ If all steps in the pipeline are succesful, a build has passed.

## Boxes
Boxes are basically virtual machines that consist of an operating system and a set of packages installed that support your stack of choice. For a while now wercker has had default **boxes** that suport the **Node.js**, **Python** and **Ruby** programming environments. In terms of services, we've had **MongoDB**, **MySQL**, **Postgres**, **Redis** and **RabbitMQ**.

You leveraged these boxes through the [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file. For instance, using the wercker default Python box:

``` yaml
box: wercker/python
```

Also for services you would use the **wercker.yml** file, as seen below for the use of the [MongoDB](http://mongodb.org) box:

``` yaml
box: wercker/python
services:
    - wercker/mongodb
```

Up until now, you were limited to using the default boxes supplied by wercker.

## Creating your own boxes

We have opened up wercker such that you are now able to create your own boxes. Creating these boxes is trivial; you can either provision these boxes through **bash** based scripting or full-fledged provisioning frameworks such as Chef or Puppet.

Similar to applications on wercker, boxes are defined through a single file called **wercker-box.yml**. Boxes can be deployed to the wercker registry, which is an index of both boxes and steps, which we will discuss in a bit.

Deploying you boxes to the wercker registry allows not only you to use your own programming language or service, but lets other developers on wercker use them as well!

To help you get started we've create a guide to creating a box with [RethinkDB](http://rethinkdb.com), an open source, distributed database, which you can view [here](http://devcenter.wercker.com/articles/boxes/bash.html).

## Steps

Alongside boxes, we've opened up the creation process for **step**