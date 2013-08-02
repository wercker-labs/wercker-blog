---
title: Creating your wercker boxes with Puppet
date: 2013-08-02
tags: boxes, puppet, opendelivery
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
  We recently introduced the <a href="http://blog.wercker.com/2013/07/22/Announcing-the-Open-Delivery-platform.html">Open Delivery platform</a> that allows you
to create your own boxes that run your wercker <a href="http://devcenter.wercker.com/articles/introduction/pipeline.html">pipeline</a> and the <a href="http://devcenter.wercker.com/articles/steps/">steps</a> that make up this pipeline.
We've also explained how to provision your wercker <a href="http://blog.wercker.com/2013/07/23/Building-your-own-box-with-Bash.html">boxes</a> with <a href="http://blog.wercker.com/2013/07/24/Building-your-own-box-with-Chef.html">Chef</a> and <a href="">Bash</a>
</h4>

![image](http://f.cl.ly/items/3P1G1Q0g0p2i2V3j3L07/wercker%2Bpuppet.png)

READMORE

In this post we will go through the process of creating a wercker box
which is provisioned via [Puppet](https://puppetlabs.com).

We assume some basic knowledge of Puppet, in order to build this box.
See the [Puppet documentation](http://docs.puppetlabs.com/) for more more information on modules and manifests.

We will also be using [librarian-puppet](https://github.com/rodjek/librarian-puppet), a bundler type tool for your puppet infrastructure.

Boxes are git repositories which are added to wercker, similar to
applications. However, instead of deploying these to the cloud, you
deploy to the [wercker directory](http://app.wercker.com/#explore).

The box we will be building will run [Memcached](http://memcached.org/), a distributed memory object caching system.

You can find the source of this box on [GitHub](https://github.com/mies/box-memcached) and its wercker page
[here](https://app.wercker.com/#applications/51f917a4f6019b7a5a0005b8/tab/details).

## Getting started

First create a git repository for your box and a file called
`wercker-box.yml` with the following contents:

``` yaml
name: memcached
version: 0.0.12
inherits: mies/librarian-puppet@0.0.2
type: service
platform: ubuntu@12.04
description: wercker box for memcached
keywords:
  - memcached
  - caching
packages:
  - memcached@1.4.13
script: |
  sudo apt-get update
  sudo puppet apply --modulepath=modules manifests/init.pp
env:
  WERCKER_MEMCACHED_PORT: 11211
  WERCKER_MEMCACHED_HOST: $$HOST$$
  WERCKER_MEMCACHED_VERSION: "1.4.13"
```

Similar to the [wercker.yml](/articles/werckeryml) file, the
`wercker-box.yml` is a definition for how your box should be
provisioned.

## The wercker-box.yml
We will be using Puppet for provisioning, but first let's go
over the abovementioned `wercker-box.yml` file.

#### Name and version

The `name` and `version` clauses are simple enough to understand.
Each time you want to deploy a new version of your box, make sure to bump the version number.

#### Inherits

We have inheritence for boxes and as such we inherit from a box called
[mies/librarian-puppet](https://github.com/mies/box-librarian-puppet). This box in turn
inherits from
[wercker/ubuntu12.04-ruby2.0.0](https://github.com/wercker/box-ubuntu12.04-ruby2.0.0) as
Puppet itself is installed via a `gem`.

With regards to inheritance, you may want to keep
your boxes lightweight. Otherwise the result could be that packages will
conflict with each other in one way or another. It also results in your
boxes being sharp, and fitted for a single purpose, as opposed to a
blunt instrument.

#### Type

Next, we specify the `type` of our box which can either be `service` or `main`. The latter is used only for pure programming language oriented boxes.

#### Platform

The `platform` clause lets us specify the underlying host operating system. Currently wercker only supports `ubuntu 12.04`.

#### Description, keywords and packages

The `description`, `keywords` and `packages` tags allow us to add
metadata to our box, which will be added to the wercker directory.
Packages will lock the version of our box in the directory as well, so
make sure to increment it when a new version of the underlying service
is available.

#### Script

In the script element we define the command(s) necessary to provision
our box. We run an apt-get update afterwards we do a **puppet apply** to install our modules.
We'll get back to how puppet gets the modules in a bit (**hint**: thanks librarian-puppet).

#### Env

These environment variable are created in the `env` clause.
We've defined env vars such as `WERCKER_MEMCACHED_HOST` to which we can
connect from our applications. Similarly we've defined the port
and the Memcached version we have installed.

## How the box gets its modules

We will use [librarian-puppet](https://github.com/rodjek/librarian-puppet) to install the modules onto the box, before we provision them in the abovementioned **wercker-box.yml**.
But how do we do this? We leverage the wercker build [pipeline](http://devcenter.wercker.com/articles/introduction/pipeline.html) to do so!

First we need to declare our puppet modules dependencies. Librarian-puppet does this through a **Puppetfile** that should look as follows:

``` ruby
mod "ntp",
  :git => "git://github.com/puppetlabs/puppetlabs-ntp.git"

mod "memcached",
  :git => "https://github.com/nixon/puppet-memcached.git"
```

In this file we declare both an NTP module and of course the Memcached module.

Now, we need a [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file like any other application on wercker, so we can install the modules declared above. Create this file with the following content:

``` yaml
box: mies/librarian-puppet@0.0.2
build:
  steps:
    - script:
        name: librarian install
        code: |
          cd $WERCKER_ROOT
          librarian-puppet install --verbose
```

Here in the **script** element we run **librarian-puppet** that installs the modules that we've defined in our **Puppetfile**.

### Adding your box to the wercker directory

Make sure to add your repository to wercker. If you need some guidance with this, see the [getting started guide for the web](/articles/gettingstarted/web.html), that will help you do so.

Add your wercker-box.yml to your git repository and push it to either GitHub or Bitbucket.

``` bash
git add .
git commit -am 'added wercker-box.yml and others'
git push origin master
```

This should trigger a build, which should pass.

You need to deploy your box to the wercker directory, just like any other application. As such, you also need to create a deploy target for the directory, so let's do so.

Go to the `settings tab` and under the *Deploy targets* section click **add deploy target**, after which a dialog is presented.

![image](http://f.cl.ly/items/25463i3b3q1y0A2e1F3i/Screen%20Shot%202013-07-22%20at%203.10.40%20PM.png)

Pick **wercker directory** as a deploy target and give it a name (such as directory).

![image](http://f.cl.ly/items/200a290x181A1f2F1F1u/Screen%20Shot%202013-07-23%20at%209.53.17%20AM.png)

Now go to your green build and hit the deploy button. Keep in mind that this deployment process can take a while as wercker is actually provisioning your box.

![image](http://f.cl.ly/items/0f0T1B0I19422i081U3v/Screen%20Shot%202013-07-22%20at%203.20.24%20PM.png)

Your box should now be available in the directory!

Explore what is currently available in the [wercker
directory](http://app.wercker.com/#explore).