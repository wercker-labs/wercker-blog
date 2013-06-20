---
title: Getting started with Node.js, Mongoose and MongoDB Part 2
date: 2013-06-21
tags: wercker, nodejs, mongodb, mongoose, tdd, javascript
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
published: false
---

<h4 class="subheader">
  This is a followup from our <a href="http://blog.wercker.com/2013/06/20/Getting-started-with-Node-Mongoose-MongoDB-Part1.html">previous post</a> where we created an application with <a href="http://nodejs.org">node.js</a>, powered by <a href="http://www.10gen.com/">10gen's</a> <a href="http://mongodb.org">MongoDB</a>. Not that we've set up our build pipeline with wercker, we are going to deploy our application to <a href="http://heroku.com">Heroku</a> and use <a href="http://mongolab.com">MongoLab</a>, a MongoDB-as-a-Service cloud platform.
</h4>

![image](http://f.cl.ly/items/18081Z1D403n3t0L0S2Y/wercker%2Bmongolab%2Bnode.jpg)

READMORE

### Introduction

In our [previous post](http://blog.wercker.com/2013/06/20/Getting-started-with-Node-Mongoose-MongoDB-Part1.html) we created a [node.js](http://nodejs.org) application powered by [express](http://expressjs.com) and [mongoose](http://mongoosejs.com), a object document mapper for [MongoDB](http://mongodb.org). We created this application using test-driven development using [Mocha](http://visionmedia.github.io/mocha/) and [SuperTest](https://github.com/visionmedia/supertest).

You can visit the finished application on wercker [here](https://app.wercker.com/#project/51c032c8b5c1c6ab300005ac).

The code to this tutorial app is [open source](https://github.com/mies/getting-started-nodejs-mongoose) on GitHub so feel free to fork and clone it.

Now, let's deploy our app!

### Deploying our app

We will deploy our application to Heroku, so make sure you have a (verified) [Heroku](http://heroku.com) account and have installed the [Heroku Toolbelt](http://toolbelt.heroku.com). Let's first create an app:

``` bash
heroku create

Creating guarded-atoll-9149... done, stack is cedar
http://guarded-atoll-9149.herokuapp.com/ | git@heroku.com:guarded-atoll-9149.git
Git remote heroku added
```
Now we need a MongoDB instance in the cloud! Fortunately, Heroku has a Marketplace (wercker is [on it](http://addons.heroku.com/wercker) by the way) filled with addons. One of which are our friends at [Mongolab](http://mongolab.com), so let's use their [add-on](https://addons.heroku.com/mongolab) to [provision](https://devcenter.heroku.com/articles/mongolab) a MongoDB database:

``` bash
heroku addons:add mongolab

Adding mongolab on guarded-atoll-9149... done, v3 (free)
Welcome to MongoLab.  Your new subscription is ready for use.  Please consult the MongoLab Add-on Admin UI for more information and useful management tools.
Use `heroku addons:docs mongolab` to view documentation.
```

For our `test` environment we used the `WERCKER_MONGODB_HOST` environment variable that was provided by wercker and was defined through the `wercker.yml`. We now need a similar environment variable for our production setting which is powered by Heroku and Mongolab. We can retrieve this environment variable, again via the Heroku command line interface:

``` bash
heroku config | grep MONGOLAB_URI

MONGOLAB_URI: mongodb://heroku_app3489u7438034:ofs0gfjfsdgsdfdsgfobfsd345ffsd3ej738@dfsdf45fsd628.mongolab.com:31628/heroku_app16434933244
```

We now need to use this environment variable, `MONGOLAB_URI`, in our actual application. Modify the **configure** section in your `app.js` file in the following way:

``` javascript
// Configure express
app.configure('development', function() {
  mongoose.connect('mongodb://localhost/todos');
});

app.configure('test', function() {
  mongoose.connect('mongodb://'+ process.env.WERCKER_MONGODB_HOST + '/todos');
});

app.configure('production', function() {
  mongoose.connect('mongodb://' + process.env.MONGOLAB_URI + '/todos');
});
```

Similarly to adding the `NODE_ENV=test` environment variable to wercker in the previous post, we need to do the same for Heroku, but now of course `NODE_ENV=production` as Heroku is our production environment.

``` bash
heroku config:set NODE_ENV=production

Setting config vars and restarting guarded-atoll-9149... done, v6
NODE_ENV: production
```
We now have succesfully set up our production environment consisting of Heroku and MongoLab.

### Creating a Heroku Procfile

Heroku needs to know which process to run on their cloud platform to actually launch your application. This is done throught the Heroku [Procfile](https://devcenter.heroku.com/articles/procfile).

Create a file called `Procfile` in your project directory with the following line of code:

```
web: node app.js
```

Let's add this file to our repository and push it to our version control system:

``` bash
git add Procfile
git commit -am 'added Procfile'
git push origin master
```

This will trigger a new build on wercker (which should pass as we didn't change anything dramatic) and we're now ready to deploy our application.

### Add deploy target

First, we add a deploy target. Go to your settings tab for your application on wercker and look for the **deploy targets** section. Although wercker has an [add-on] on the Heroku Marketplace, making deployment even easier, we are going to add Heroku manually as a deploy target.

![image](http://f.cl.ly/items/352B0a0W1Y0b1b0n0W0I/Screen%20Shot%202013-06-20%20at%2012.32.30%20PM.png)

As the page indicates, retrieve your Heroku API key from your [Heroku Dashboard](https://dashboard.heroku.com/account) and paste in the form. Next, you are presented with a form where you can *name* your deploy target (let's go with *production*) and are able to auto deploy specific branches. We've previously written a post on auto-deployment [here](http://blog.wercker.com/2013/06/05/Autodeployment.html). We are also able to either select an existing Heroku app that we want to deploy to, or create one. I'm going to pick the application I've previously created using the `heroku create` command and to which I've also added the [MongoLab add-on](https://addons.heroku.com/mongolab)

![image](http://f.cl.ly/items/2u1r3F2T3t2F2i2q2Q38/Screen%20Shot%202013-06-20%20at%2012.35.08%20PM.png)

Let's deploy our application!

### Deploying our application

Go to the latest green build on wercker and hit the *deploy this build* button. A pull down menu will appear and you can select the target ('production') that you've just created,

![image](http://f.cl.ly/items/2m151d1l3i1o3V3d3P3L/Screen%20Shot%202013-06-20%20at%201.29.14%20PM.png)

If you go to the **deploys** tab you can see your deploy in action:

![image](http://f.cl.ly/items/3q3d13263H3r2k2h3w1Y/Screen%20Shot%202013-06-20%20at%201.24.59%20PM.png)

If we visit the root of our application on Heroku we can see it in action:

![image](http://f.cl.ly/items/2u0D3m062M40032z0r1Y/Screen%20Shot%202013-06-20%20at%201.55.05%20PM.png)

Congratulations! You've succesfully built a REST API using test-driven development and leveraged wercker for continuous delivery. Subscribe to the [feed](http://blog.wercker.com/feed.xml) of our blog and follow us on [twitter](http://twitter.com/wercker) for more updates.

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Don't forget to sign up for wercker for free [here](https://app.wercker.com/users/new/).