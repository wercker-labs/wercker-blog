---
title: Compass and Sass compilation on wercker
date: 2013-07-12
tags: compass, sass, pipeline, buildsteps
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
In this tutorial we will briefly discuss how to set up your sass and compass compilation steps on wercker. <a href="http://sass-lang.com/">Sass</a> is a language on top of CSS enabling you to add more structure to your stylesheets. <a href="http://compass-style.org/">Compass</a> is a framework that makes it easier to author your Sass stylesheets.
</h4>

Wercker can not only be leveraged to run your unit test, but also to build and compile static assets. This includes for instance the minification of javascript files or as this article will describe, compilation of stylesheets.

![image](http://f.cl.ly/items/3633152I261P0f2u0N0f/Sass_Logo.gif)

You can find the code for this tutorial on GitHub [here](https://github.com/mies/getting-started-sass) and view it on wercker by clicking on my build-status badge below:

[![Wercker status](https://app.wercker.com/status/be7bd3a159ec607d1ba67896bb641e4a/m)](https://app.wercker.com/project/bykey/be7bd3a159ec607d1ba67896bb641e4a)

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).

READMORE

## Build Steps

Each build on wercker consists of steps that you can create yourself by declaring these steps in the [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file. In this tutorial we are going to set up our **wercker.yml** file to execute a compass compilation for us on Sass stylesheets.

First, make sure you have a `git` repo on your machine and GitHub or Bitbucket.
You can fork my repo [here](https://github.com/mies/getting-started-sass), that already contains the code for the tutorial.

## Creating your project

First, declare your dependencies in your `Gemfile`:

``` ruby
source 'https://rubygems.org'
gem 'compass'
```

We just need the [Compass](http://compass-style.org/) framework in our case. Next, bootstrap your compass project, by running `compass create`:

``` bash
compass create

directory sass/
directory stylesheets/
   create config.rb
   create sass/screen.scss
   create sass/print.scss
   create sass/ie.scss
   create stylesheets/ie.css
   create stylesheets/print.css
   create stylesheets/screen.css

*********************************************************************
Congratulations! Your compass project has been created.

```

Compass has created several files for us, we can [.gitignore](http://git-scm.com/docs/gitignore) the **stylesheets** folder and the ***.sass-cache** as we will be running our compass compilation on wercker.

``` bash
echo 'stylesheets/\n.sass-cache' > .gitignore
```


Lets do some basic editing of sass stylesheets. Open up **screen.scss** with your favorite editor and create some styles:

``` css
@import "compass/reset";

/* screen.scss */
#navbar {
  width: 80%;
  height: 23px;
}

a {
  color: #ce4dd6;
  &:hover { color: #ffb3ff; }
  &:visited { color: #c458cb; }
}
```

We now have some basic styling in place!

## Adding our project to wercker

Now we're going to set up our build pipeline on wercker. Create a file called `wercker.yml`. You can read more about setting up your wercker.yml file on our [dev center](http://devcenter.wercker.com/articles/werckeryml/).

``` yaml
box: wercker/ubuntu12.04-ruby2.0.0
build:
  steps:
    - bundle-install
    - script:
        name: compile compass assets
        code: bundle exec compass compile
```

Lets go over this file. The box element tells wercker what kind of execution environment our project needs. In this case we want to have Ruby version 2.0 available. You can read more about the [wercker pipeline](http://devcenter.wercker.com/articles/introduction/pipeline.html) and boxes on the wercker [dev center](http://devcenter.wercker.com/articles/boxes/)

Next we define our build pipeline using steps. We need to install our dependencies first, so we execute the by wercker provided `bundle-install` step for Ruby projects. Now we create a custom step that we've named **compile compass assets** and that runs the `compass compile` command within the **bundled** environment.

## Push to git

Now add your **sass** folder, the by compass created **config.rb**, the **Gemfile** and the **wercker.yml** file to git.

``` bash
git add .
git commit -am 'init'
git push origin master
```

## Add your project to wercker

Now you want to add your project to wercker. You can do so either via the [web interface](http://devcenter.wercker.com/articles/gettingstarted/web.html) or through the [wercker command line interface](http://devcenter.wercker.com/articles/gettingstarted/cli.html). We will use the CLI (read more on the CLI [here](http://devcenter.wercker.com/articles/cli/) in this example, that we can install via `pip`:

``` bash
pip install wercker
```

Now run the following command in your project folder to add your application to wercker:

``` bash
wercker create
```

The CLI will detect that you already have a [git remote](http://gitref.org/remotes/) on GitHub or Bitbucket and use it to set up your application on wercker. It wil also trigger an initial build.

You can view your build on wercker by running:

``` bash
wercker open
```

You will now see your compass compilation step that we added to the **wercker.yml** file.

![image](http://f.cl.ly/items/380m1u2o3z033q1Z1l1i/Screen%20Shot%202013-07-12%20at%2012.49.18%20PM.png)

As a followup, we could deploy these stylesheets to S3, for instance via the [S3sync step](https://github.com/wercker/step-s3sync) that we've also described in a previous article on streamlining [Jekyll](http://blog.wercker.com/2013/05/31/simplify-you-jekyll-publishing-process-with-wercker.html) and [Middleman](http://blog.wercker.com/2013/06/10/Streamlining-Middleman-Deploys-to-s3.html) assets.

## Earn some stickers

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.

Sign up for wercker [here](https://app.wercker.com/users/new/)