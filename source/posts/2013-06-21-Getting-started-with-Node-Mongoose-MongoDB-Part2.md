---
title: Getting started with Node.js, Mongoose and MongoDB Part 2
date: 2013-06-21
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


### Deploying our app

We will deploy our app to Heroku, so make sure you a [Heroku](http://heroku.com) account and have installed the [Heroku Toolbelt](http://toolbelt.heroku.com). Let's first create an app:

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
