---
title: Getting started with Node.js, Mongoose and MongoDB Part 2
date: 2013-06-21
tags: wercker, nodejs, mongodb, mongoose, tdd, javascript
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
published: false
---

<h4 class="subheader">
  This is a followup from our previous post where we created an application with <a href="http://nodejs.org">node.js</a>, powered by <a href="http://www.10gen.com/">10gen's</a> <a href="http://mongodb.org">MongoDB</a>. Not that we've set up our build pipeline with wercker, we are going to deploy our application to <a href="http://heroku.com">Heroku</a> and use <a href="http://mongolab.com">MongoLab</a>, a MongoDB-as-a-Service cloud platform.
</h4>

![image](http://f.cl.ly/items/18081Z1D403n3t0L0S2Y/wercker%2Bmongolab%2Bnode.jpg)

READMORE

### Introduction

In our [previous post](http://blog.wercker.com/2013/06/20/Getting-started-with-Node-Mongoose-MongoDB-Part1.html) we created a [node.js](http://nodejs.org) application powered by [express](http://expressjs.com) and [mongoose](http://mongoosejs.com), a object document mapper for [MongoDB](http://mongodb.org). We created this application using test-driven development using [Mocha](http://visionmedia.github.io/mocha/) and [SuperTest](https://github.com/visionmedia/supertest). Now, let's deploy our app!

### Deploying our app

We will deploy our application to Heroku, so make sure you have a (verified) [Heroku](http://heroku.com) account and have installed the [Heroku Toolbelt](http://toolbelt.heroku.com). Let's first create an app:

``` bash
heroku create

Creating guarded-atoll-9149... done, stack is cedar
http://guarded-atoll-9149.herokuapp.com/ | git@heroku.com:guarded-atoll-9149.git
Git remote heroku added
```
Now we need a MongoDB instance in the cloud! Fortunately, Heroku has a Marketplace (wercker is [on it](http://addons.heroku.com/wercker) by the way) filled with addons. One of which are our friends at [Mongolab](http://mongolab.com), so let's use their add-on to provision a MongoDB database:

``` bash
heroku addons:add mongolab
```

For our `test` environment we used the `WERCKER_MONGODB_HOST` environment variable that was provided by wercker and was defined through the `wercker.yml`. We now need a similar environment variable for our production setting which is powered by Heroku and Mongolab. We can retrieve this environment variable, again via the Heroku command line interface:

``` bash
heroku config | grep MONGOLAB_URI
```
Let's use this environment variable in our actual application. Modify your `app.js` file in the following way:

``` javascript
```
### add deploy target

### Conclusion

Don't forget to sign up for wercker for free [here](https://app.wercker.com/users/new/).
