---
title: Continuous Delivery for Meteor apps
date: 2013-07-19
tags: meteor, javascript, boxes
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
In this post we're going to discuss how to add continuous delivery
support to <a href="http://meteor.com">Meteor</a> apps. Meteor is an open source framework for creating
web applications at a rapid pace.
</h4>

At wercker we're all about increasing **#developervelocity**, so we love
what the guys at Meteor have built with their framework. Creating Meteor
apps is not only simple and fast, it is also tremendous fun.

In this article we will briefly discuss how to do implement continuous
delivery with wercker and the [Laika](http://arunoda.github.io/laika/)
testing framework.

![image](http://f.cl.ly/items/0y1f3B2o0Z0m0h193P10/meteorjs.jpeg)

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).

You can find the source code for this application on [GitHub](https://github.com/mies/getting-started-meteor), so feel
free to fork away! This application is public on wercker as well, so you
can view the current [build status](https://app.wercker.com/#project/51e2d1fcaabf671f79003ad7/).

[![Wercker status](https://app.wercker.com/status/e3922bee45ba7e3b08dba4997c00a0c9/m)](https://app.wercker.com/project/bykey/e3922bee45ba7e3b08dba4997c00a0c9)

READMORE

## Creating our application

Let's start out with creating our application. We assume you have Meteor
installed, read how to do so at the excellent Meteor [documentation
site](http://docs.meteor.com).

``` bash
meteor create getting-started-meteor
```

For our app we're going to create a very simple collection that will
hold todos. Create a filed called `collections.js` with the following
contents:

``` javascript
Todos = new Meteor.Collection('todos');
```

Next create a unittest called `todos.js` in a tests folder which you
need to create.

``` javascript
var assert = require('assert');

suite('Todos', function() {
  test('Metor server side', function(done, server) {
    server.eval(function() {
      Todos.insert({name: 'try out Meteor!'});
      var docs = Todos.find().fetch();
      emit('docs', docs);
    });

    server.once('docs', function(docs) {
      assert.equal(docs.length, 1);
      assert.equal(docs[0].name, 'try out Meteor!');
      done();
    });
  });
});
```

This unittest only evaluates Meteor's [server
side](http://docs.meteor.com/#structuringyourapp). We insert a new
todo with the name **try out Meteor!** and assert if our collection has
a lenght of 1 and if this one document's name equals our just inserted
Todo.

We are now ready to define our [wercker pipeline](http://devcenter.wercker.com/articles/introduction/pipeline.html).

## Setting up our wercker.yml

The [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file defines our build and deployment pipeline on
wercker. You can read more on this file at our [dev
center](http://devcenter.wercker.com).

Wercker **pipelines** run insided [boxes](http://devcenter.wercker.com/articles/boxes/), which are basically VM's preloaded
with a specific stack and environment. In this case we want to use a box
that is suited for building, testing and deploying Meteor applications.

Luckily we have created such a box and have [open
sourced](http://github.com/mies/box-meteor) it as well. This meteor box also
contains [phantomjs](http://phantomjs.org), which Laika uses.

In your repository
create a file called wercker.yml with the following contents:

``` yaml
box: mies/meteor
build:
  steps:
    - script:
        name: echo nodejs information
        code: |
          echo "node version $(node -v) running"
          echo "npm version $(npm -v) runnin:g"
    - script:
        name: install laika
        code: sudo npm install -g laika
    - script:
        name: run laika
        code: laika -D

```

As you can see, I have selected the `mies/meteor` box for my pipeline,
that I mentioned above.

This wercker pipeline consists of three [build](http://devcenter.wercker.com/articles/introduction/builds.html) [steps](http://devcenter.wercker.com/articles/steps/). The first just echos
back the Node and Npm version. The second actually installs Laika and
the last step runs our unit test.

Now you are ready to add your application to wercker. You can do so via
the [command line
interface](http://devcenter.wercker.com/articles/gettingstarted/cli.html)
that you can install via `pip install
wercker`, after which you can run `wercker create`. Read more about the
CLI [here](http://devcenter.wercker.com/articles/cli/).

You can also add your application [via the web
interface](http://devcenter.wercker.com/articles/gettingstarted/web.html), of which we've
depicted a screenshot below.

![image](http://f.cl.ly/items/3s151d3a393F3A2E2g1A/Screen%20Shot%202013-07-19%20at%201.57.22%20PM.png)

Each push you do after creating your application on wercker will trigger
a build. My unit test passes, which you can see below, so now we're
ready to deploy our application to [Heroku](http://heroku.com).

![image](http://f.cl.ly/items/1t3p0Y230V143w0X1N1B/Screen%20Shot%202013-07-19%20at%202.01.29%20PM.png)

## Deploying to Heroku

Create an application on Heroku using the [Meteor
buildpack](https://github.com/jordansissel/heroku-buildpack-meteor)

``` bash
heroku create --stack cedar --buildpack https://github.com/jordansissel/heroku-buildpack-meteor.git
```

Now configure your ROOT_URL setting, which is needed for Meteor apps
running on Heroku:

``` bash
heroku config:add ROOT_URL=<insert_url_created_from_heroku_create_command>
```

## Add a deploy target on wercker

Though wercker has an [add-on](http://devcenter.wercker.com/articles/deployment/heroku.html) on the [Heroku
Marketplace](https://addons.heroku.com/wercker) we will add our Heroku
deploy target in a manual way.

Go to you application's **Settings page** on wercker and find the
section labeled **Deploy target**. Click add deploy target and pick
Heroku. Wercker will ask for your API key which you can find on your
[Heroku account page](https://dashboard.heroku.com/account). Add your
key and select the application you just created via the **heroku
create** command, as seen below.

![image](http://f.cl.ly/items/2o2N232L1u471k1j1Q1i/Screen%20Shot%202013-07-19%20at%202.15.26%20PM.png)

Fill in a name for your deploy target such as **staging** or
**production**. You can choose to autodeploy your builds as well.

## Deploy your application

Now go to your latest green build on wercker and click the **Deploy this
build** button.

![image](http://f.cl.ly/items/142i1g1S3x0P2R2b2t2z/Screen%20Shot%202013-07-19%20at%202.51.41%20PM.png)

This will trigger a build and add a deploy key to your Heroku account.

![image](http://f.cl.ly/items/0u2w0u2Z1M3K3a441t1z/Screen%20Shot%202013-07-19%20at%202.51.01%20PM.png)

If you now visit your Meteor application on Heroku, you'll see that its live
and running!

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.




