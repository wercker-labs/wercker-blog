---
title: Using Node.js 0.10.x on wercker
date: 2013-06-28
tags: wercker, nodejs, javascript, development
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
In several <a href="http://blog.wercker.com/2013/06/20/Getting-started-with-Node-Mongoose-MongoDB-Part1.html">previous</a> <a href="http://blog.wercker.com/2013/06/21/Getting-started-with-Node-Mongoose-MongoDB-Part2.html">posts</a> we've mentioned that team wercker is a big fan of Node.js and some of the wercker components are written in Node. We've now added support for Node.js version 0.10.x and in this post we briefly want to discuss setting up your Node.js 0.10.x projects.
</h4>

READMORE

### wercker.yml

Before we get started; signing up for wercker is [free and easy](https://app.wercker.com/users/new/).

The [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file is the way to set up your build and deployment pipeline on wercker, it is a simple [dsl](http://en.wikipedia.org/wiki/Domain-specific_language), written in [yaml](http://www.yaml.org/).

The default version of Node.js on wercker is currently at `0.8.24` but by changing our box definition we can make use of version `0.10.11`. Your wercker.yml should look as follows:

``` yaml
box: wercker/ubuntu12.04-nodejs0.10
# Build definition
build:
  # The steps that will be executed on build
  steps:
    # A step that executes `npm install` command
    - npm-install
    # A step that executes `npm test` command
    - npm-test

    # A custom script step, name value is used in the UI
    # and the code value contains the command that get executed
    - script:
        name: echo nodejs information
        code: |
          echo "node version $(node -v) running"
          echo "npm version $(npm -v) running"
```

The most important line here is:

``` yaml
box: wercker/ubuntu12.04-nodejs0.10
```

That's it! We'll be making Node.js version 0.10.x the default pretty soon (but you'll still be able to revert to older versions, again through the box definition), so make sure to follow our [blog](http://blog.wercker.com) or [twitter](http://twitter.com/wercker).

I have created a separate branch of our sample application called 'getting-started-nodejs' that uses Node.js 0.10.x [here](https://github.com/mies/getting-started-nodejs/tree/node-0.10.x).

You can also view its status and build pipeline on [wercker](https://app.wercker.com/#project/51c02591b5c1c6ab300002a0).

### Earn some stickers!

Tell us about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy]((https://app.wercker.com/users/new/).