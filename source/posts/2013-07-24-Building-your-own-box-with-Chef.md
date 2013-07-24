---
title: Building your own CouchDB box with Chef
date: 2013-07-24
tags: opendelivery, boxes, chef, couchdb
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
In this article we will go through the process of creating a <a href="http://devcenter.wercker.com/articles/boxes/">wercker box</a>
which is provisioned via <a href="http://www.opscode.com/chef/">Opscode's Chef</a>. We recently introduced the <a href="http://blog.wercker.com/2013/07/22/Announcing-the-Open-Delivery-platform.html">Open Delivery platform</a> that allows you
to create your own boxes that run your wercker <a href="http://devcenter.wercker.com/articles/introduction/pipeline.html">pipeline</a> and the <a href="http://devcenter.wercker.com/articles/steps/">steps</a> that make up this pipeline.
</h4>

![image](http://f.cl.ly/items/1s3i1l2p373c1f062t1F/wercker%2Bchef.png)

READMORE

Some basic knowledge on Chef is necessary to build your own boxes with Chef.
See the [Chef documentation](http://docs.opscode.com/) for more more information on cookbooks and chef-solo.

Boxes are git repositories which are added to wercker, similar to
applications. However, instead of deploying these to the cloud, you
deploy to the [wercker directory](http://app.wercker.com/#explore).

The box we will be building will run [CouchDB](http://couchdb.apache.org/), a document oriented
database.

You can find the source of this box on [GitHub](https://github.com/mies/box-couchdb) and its wercker page
[here](https://app.wercker.com/#project/51cace444b940c9e19004ba2).

## Getting started

First create a git repository for your box and a file called
`wercker-box.yml` with the following contents:

``` yaml
name: couchdb
version: 0.0.9
inherits: mies/erlang@0.0.2
type: service
platform: ubuntu@12.04
description: wercker box for couchdb a document oriented database
keywords:
  - couchdb
  - erlang
  - noqsql
  - database
packages:
  - couchdb@2.4.0
script: |
  sudo chef-solo -c $WERCKER_SOURCE_DIR/solo.rb -j $WERCKER_SOURCE_DIR/solo.json -l debug
env:
  WERCKER_COUCHDB_PORT: 5984
  WERCKER_COUCHDB_HOST: $$HOST$$
  WERCKER_COUCHDB_URL: $$HOST$$:5984
  WERCKER_COUCHDB_VERSION: "2.4.0"
```

Similar to the [wercker.yml](http://devcenter.wercker.com/articles/werckeryml) file, the
`wercker-box.yml` is a definition for how your box should be
provisioned.

## The wercker-box.yml
We will be using Chef for provisioning, but first let's go
over the abovementioned `wercker-box.yml` file.

#### Name and version

The `name` and `version` clauses are simple enough to understand.
Each time you want to deploy a new version of your box, make sure to bump the version number.

#### Inherits

We have inheritence for boxes and as such we inherit from a box called
[mies/erlang](https://github.com/mies/box-erlang). This box in turn
inherits from
[ubuntu12.04-chef10.18.2](https://github.com/wercker/box-ubuntu12.04-webessentials) which is
a box loaded with the Chef provisioning system, that we will use to
provision the CouchDB box. With regards to inheritance, you may want to keep
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
our box. As we are using chef-solo, we just execute this command with
the **solo.json** parameter, which defines the recipes for chef.

#### Env

These environment variable are created in the `env` clause.
We've defined env vars such as `WERCKER_COUCHDB_HOST` to which we can
connect from our applications. Similarly we've defined the port, connection
url and the CouchDB version we have installed

You can explore the wercker directory for boxes [here](http://app.wercker.com/#explore).

### Setting up your cookbooks

For Chef provisioning we use [Berkshelf](http://berkshelf.com/), a cookbook dependency management
tool. We create a file called `Berksfile` that declares our cookbook dependencies:

``` ruby
site :opscode

cookbook 'build-essential'
cookbook 'apt', '1.10.0'
cookbook 'couchdb'
```

Next, the `solo.json` file is used to hold any configuration info and the runlist for the cookbooks. We need to let CouchDB bind to any inferface, as wercker services run on separate VM's and we need to be able to connect to them from any other VM that would run our application code.

``` javascript
{
  apt: {
    mirror: "us-east-1.ec2"
  },
  run_list: [
    "recipe[apt]",
    "recipe[build-essential]",
    "recipe[couchdb]"
    ],
  couch_db: {
    config: {
      httpd: {
        bind_address: "0.0.0.0"
      }
    }
  }
}
```

We also create a **solo.rb** that specifies the cookbook path. This file is pretty much the same for every wercker box unless you want a different folder to hold your cookbooks:

``` ruby
root = File.absolute_path(File.dirname(__FILE__))
file_cache_path root
cookbook_path root + '/cookbooks'
```

See the [Chef documentation](http://docs.opscode.com/) for more more information on cookbooks and chef-solo.

### Creating our wercker.yml

In order to automate the build process of our box we create a build pipeline on wercker (yes, we're building our wercker box with wercker). Like any other application on wercker, we do so through the [wercker.yml](http://devcenter.wercker.com/articles/werckeryml) file:

``` yaml
box : wercker/ubuntu12.04-ruby1.9.3-berkshelf1
build :
  steps :
    - script:
        name : install cookbooks
        code : |
          cd $WERCKER_ROOT
          berks install -p cookbooks
```

We inherit from the **berkshelf** box and add a buildstep that runs the **berks** command installing our cookbooks.

### Adding your box to the wercker directory

Make sure to add your repository to wercker. If you need some guidance with this, see the [getting started guide for the web](http://devcenter.wercker.com/articles/gettingstarted/web.html), that will help you do so.

Add your wercker-box.yml to your git repository and push it to either GitHub or Bitbucket.

``` bash
git add wercker-box.yml
git commit -am 'added wercker-box.yml'
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

You can check out my CouchDB box [here](https://app.wercker.com/#applications/51cace444b940c9e19004ba2/tab/details)

Explore what is currently available in the [wercker directory](http://app.wercker.com/#explore).

![image](http://f.cl.ly/items/3A153L0J1Z1i0t081V0N/Screen%20Shot%202013-07-23%20at%2010.00.35%20AM.png)


## Earn some stickers!

Let us know about the boxes you create for wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.