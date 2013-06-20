---
title: Getting started with Node.js, Mongoose and MongoDB Part 1
date: 2013-06-20
tags: wercker, nodejs, mongodb, mongoose, tdd, javascript
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
published: false
---

<h4 class="subheader">
  The wercker technology stack consists of <a href="http://nodejs.org">node.js</a> and is powered by <a href="http://www.10gen.com/">10gen's</a> <a href="http://mongodb.org">MongoDB</a>, so it's
  about time that we feature a blog post on building and testing a node and mongo application with wercker! So let's get started!
</h4>

![image](http://f.cl.ly/items/0x3f1I3h00140n200j34/wercker%2Bmongodb%2Bnode.jpg)

READMORE

### Introduction

In this tutorial we are going to build a complete [REST API](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) consisting of node.js, mongodb; tested and delivered by wercker. This tutorial is also available in shortform on our [dev center]().
We will be leveraging the [express](http://expressjs.com/) and [mongoose](http://mongoosejs.com/) packages available on [npm](http://npmjs.org). The application that we will be building will be an API that exposes and stores **todo items**. We will be using [MongoDB](http://mongodb.org) to store our items, which is a powerful and flexibel non-relational datastore.

For your convenience you can fork and clone the finished repository from [GitHub](https://github.com/wercker/getting-started-nodejs-mongoose). You can visit my finished application on wercker with its current buildstatus [here](https://app.wercker.com/#project/51c032c8b5c1c6ab300005ac).

You can sign up for wercker for free [here](https://app.wercker.com/users/new/).

### Declaring our dependencies

Let us first declare our dependencies through the `package.json` format. We will be using [express](http://expressjs.com/) as our application framework, and [mongoose](mongoosejs.com), an object document mapper for MongoDB and node.js applications. For testing purposes we will leverage [Mocha](http://visionmedia.github.io/mocha/) and [SuperTest](https://github.com/visionmedia/supertest).

We also immediately include a `script` clause that calls `npm test` so our tests will be run. Our final `package.json` file looks as follows:

``` javascript
{"name": "getting-started-nodejs-mongoose",
  "description": "Sample application built with node.js, mongoose, mongodb. Delivered by wercker.",
  "version": "0.0.1",
  "private": true,
  "dependencies": {
    "express": "3.x",
    "mongoose": "3.6.x",
    "supertest": "0.7.0",
    "mocha": "1.11.0"
  },
  "scripts": {
    "test" : "mocha",
    "start": "app.js"
  }
}
```

### Defining our wercker.yml

The [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file declares our build and deployment pipeline. We also use it to define any services, in this case [MongoDB](http://mongodb.org), we might need. Read more on services at our [dev center](http://devcenter.wercker.com/articles/services/). Make sure your `wercker.yml` looks as follows:

``` yaml
box: wercker/nodejs
services:
  - wercker/mongodb
build:
  # The steps that will be executed on build
  steps:
    # A step that executes `npm install` command
    - npm-install
    # A step that executes `npm test` command
    - npm-test
```
Note that we use the `node.js` box and, as said, make use of `wercker/mongodb` that provides us with the `WERCKER_MONGODB_HOST` environment variable that we can use as our connection string later on. As mentioned, visit our [dev center](http://devcenter.wercker.com/articles/services/) for more information on services and the [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/).

### Writing our tests

As we're doing test-driven-development, we are going to write our tests first. We will test three routes that we will create in the next step. Specifically, we want to test the creation of **todo** items, the listing of **all** todo items and the details of a **single** todo item based on the author information provided in the request. As said, we will use [Mocha](http://visionmedia.github.io/mocha/) and [SuperTest](https://github.com/visionmedia/supertest) for the writing of our tests. In a separate `test` folder, create a file called `test.js` with the following contents:

``` javascript
```
In this file, we see the three separate use-cases that we want to test, now that we know what the results should be of our API, let's go ahead and write it.

### Creating our API

Now, let's write our API. Create a file called `app.js` that looks as follows, we will go over it a bit.

``` javascript
```

We define our, `dev`, `production` and `testing` environments with any specifcs we might need. For instance, our development environment might have a local instance of MongoDB running, whereas in our `test` and `production` environments these settings obviously differ. In the case of wercker, we utilize the environment variable, `WERCKER_MONGODB_HOST`, provided by defining our dependency on MongoDB in the previously defined `wercker.yml` file.

#### Defining our Mongoose Model

We also need to create a model object that will define what a **todo item** actually looks like, and what kind of attributes it has.

Create a file called `todo.js` in a folder named `models` with the following contents:

``` javascript
var mongoose = require('mongoose');
var Schema = mongoose.Schema;
var ObjectId = Schema.ObjectId;

var todoSchema= new Schema({
  action: String,
  author: String,
  creationDate: {type: Date, 'default': Date.now},
});

module.exports = mongoose.model('Todo', todoSchema);

```
A **todo** item has an *action* (the todo!), an author (who created it) and a creation date that defaults to the date when an item was inserted in the database.

Now that our application, models and tests are complete, we are ready test and deploy it with wercker.

### Adding our application to wercker

Sign in to wercker and click on the `add application` button. Make sure you have your repo on GitHub and have connected GitHub to wercker, as showcased below.

![image](http://f.cl.ly/items/1k0l1o1g31063x1r020X/Screen%20Shot%202013-06-19%20at%204.37.39%20PM.png)

#### Create a build environment variable

We have previously defined a `dev`, `test` and `production` setting in our application. The express framework uses the `NODE_ENV` environment variable to determine which setting to pick. For wercker we need to use the `test` setting, so how do we set this up? In wercker you can set build specific environment variables. In the **settings tab** of your application go to the section that reads **build**. Here we can set the `NODE_ENV` variable with the value `test` as shown below:

![image](http://f.cl.ly/items/1X172r3h2k272A1Y3h2S/Screen%20Shot%202013-06-19%20at%206.00.25%20PM.png)

After you've finished adding your application, we are ready to push our repository.

``` bash
git add .
git commit -am 'init'
git push origin master
```

This will trigger a build on wercker which looks like this:

![image](http://f.cl.ly/items/3h1K2f3g1S242f3X0o1T/Screen%20Shot%202013-06-19%20at%204.46.42%20PM.png)

If the build passed, we're ready to deploy our application, which we will discuss in Part 2 of this post, so stay tuned.

Don't forget to sign up for wercker for free [here](https://app.wercker.com/users/new/).
