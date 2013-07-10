---
title: Creating a RethinkDB box for wercker
date: 2013-07-11
tags: rethinkdb, boxes
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
published: false
---

<h4 class="subheader">
In this post we describe how to create a box that runs <a
href="http://rethinkdb.com">RethinkDB</a> for wercker and deploy it
to the <a href="">just announced</a> wercker registry.
</h4>

READMORE

Wercker [boxes](http://devcenter.wercker.com/articles/boxes/) are lightweight VM's that run your [build] and [deploy]
pipeline on wercker. Boxes define the stack (programming environment,
databases, queues etc) that you want to leverage for your continuous
delivery process.

By default you can use several [supported stacks] and
[services](http://devcenter.wercker.com/articles/services), hence boxes, within
wercker.

We recently announced the [wercker registry] an index of boxes and
[steps]
that you can not only use for your projects on wercker, but also
contribute to.

In this article we will go into detail on creating a box running
RethinkDB, an open source, distributed database.

## wercker-box.yml

Similar to defining your build and deploy pipeline with the
[wercker.yml](http://devcenter.wercker.com/werckeryml/) file, boxes are
created via a [wercker-box.yml]().

Boxes can be provisioned either via Bash or [Chef], but it is fairly
easy to roll your own (we'd love to see boxes being created via Puppet or
Ansible!).

Box definitions are stored in git and by extension, are wercker
applicaitons. Instead of deploying your box (**an application**) to your
cloud provider, you now deploy it to the [wercker registry]

The format for the `wercker-box.yml` is pretty simple so let's go over
it
