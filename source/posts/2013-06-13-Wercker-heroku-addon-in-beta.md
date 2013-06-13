---
title: Wercker Heroku addon now in beta
date: 2013-06-13
tags: wercker, heroku, addon, cloud, paas
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
Wercker is a big fan of what Heroku has accomplished in terms of increasing developer velocity with their amazing cloud platform. Heroku has also empowered developers to create other cloud services on top of their platform through the Heroku Marketplace. This article deals with the wercker Heroku add-on, now available in <a href="https://addons.heroku.com/wercker">beta</a>.
</h4>

READMORE

### Signing up for the Heroku beta program
As the wercker addon is in beta, it is only available to members of the Heroku beta program, for which you can register up [here](https://beta.heroku.com/). Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).

##### note: you can also deploy to heroku by creating a [deploy target](http://devcenter.wercker.com/articles/introduction/deployment.html) and your Heroku api key
****

## How does it work?

Visually, the flow for wercker is as follows:

![image](http://f.cl.ly/items/24352w223K2v142I1Y1V/heroku_flow.jpg)

Wercker connects with your GitHub or Bitbucket repository. Each time you do a `git push` wercker receive a signal that new code has been created and wil subsequently start running your tests in a sand-boxed environment. If your build is green, you are ready to deploy your application to Heroku. Through the Heroku add-on you get a concise dashboard with an overview of your builds and deploys.

##### NOTE: Having a separate repository on a version control platform such as GitHub is a requirement of using wercker. It is also good practice.
****


## Requirements

- A wercker account. Sign up [here](https://app.wercker.com/users/new/)
- A [Heroku](http://heroku.com) account and the [Heroku toolbelt](http://toolbelt.heroku.com)
- A remote git repository for your application. Wercker supports [GitHub](http://github.com) and [Bitbucket](http://bitbucket.org).
- Python and [pip](http://www.pip-installer.org/) to install the wercker command-line interface


## Creating your application
First we need an application that we want to build and deploy with wercker. For your convenience we've already created an application in Ruby using
the [sinatra](http://sinatrarb.com) micro framework. Take a look on the
[werkcer GitHub page](https://github.com/wercker/getting-started-ruby) and go ahead and fork
this repository and then clone it:

``` bash
git clone git@github.com:<YOUR USERNAME HERE>/getting-started-ruby.git
```

Go into your repository folder and you will see, that indeed it is a simple
sinatra API that returns a collection of city names in JSON. It also has
a unit test, created with [rspec](http://rspec.info/).

The repository also contains a `Procfile` which we will use to deploy
our application to Heroku.

Now that we have an application lets create it on Heroku:

``` bash
$ heroku create
Creating stark-fog-398... done, stack is cedar
http://stark-fog-398.herokuapp.com/ | git@heroku.com:stark-fog-398.git
Git remote heroku added
```

## Provisioning the add-on
Wercker can be added to your application using the Heroku CLI:

``` bash
$ heroku addons:add wercker
Adding wercker on stark-fog-398... done, v7 (free)
Use `heroku addons:open wercker` to get started.
Use `heroku addons:docs wercker` to view documentation.
```


Next, open the wercker wizard by running the following command:

    $ heroku addons:open wercker

This wizard will guide you through the steps needed to run your first build on wercker and looks as follows:

![image](http://f.cl.ly/items/1s0r1b42003x2K1y2T3S/Screen%20Shot%202013-06-13%20at%202.39.01%20PM.png)

## Local Setup

As you can guess from the screenshot of the wizard, wercker comes with a command line interface that you can install by running:

    $ pip install wercker

You might need to run this as `sudo pip install wercker` depending on your platform.


The CLI helps you interact with the wercker platform. Run the following command to link your application with wercker:

    $ wercker create

You will receive the following response

    -----------------------
    welcome to wercker-cli
    -----------------------

    Searching for git remote information...
    Found 1 repository location(s)...

    Please choose one of the following options:
     (1) git@github.com:mies/ruby-sample.git
    Make your choice (1=default):

As mentioned above, if you now run `heroku addons:open wercker` or go to the Heroku dashboard and click on the wercker resource, you will see the wercker wizard that will guide you through your build and deploy.

##### NOTE: The CLI will also ask you to add the `werckerbot` user to your repo. In order to run your tests `werckerbot` needs read permisson on either GitHub or Bitbucket. Read more on werckerbot [here](http://devcenter.wercker.com/articles/gettingstarted/werckerbot.html)
****

The CLI will also detect your Heroku remote and present the following feedback:


``` bash
Step 2.
-------------

1 automatic supported target(s) found.
Heroku remote git@heroku.com:shrouded-wave-4805.git selected.
Heroku deploy target shrouded-wave-4805 successfully added to the wercker application
````

It automatically adds your Heroku application as a [deploy target](http://devcenter.wercker.com/articles/introduction/deployment.html) to wercker.


## Your first build and deploy

If all went well your first build is triggered after you've succesfully run `wercker create`. Upon any subsequent `git push` commands wercker gets triggered and will run your a build. You will also be able to add any teammembers you might have for this application.

You are now ready to deploy your build to Heroku, if it passed of course. You can deploy your build in two ways:

- Through the command line interface via the `wercker deploy` command. The CLI will ask which build you want to deploy to which target (you could after all, have multiple Heroku applications)
- Through the wercker wizard. As said, running `heroku addons:open wercker` will open the dashboard and you are now ready initiate your first deploy. See below:

![image](http://f.cl.ly/items/150j0P2L1x1Q2w1E3o3U/Screen%20Shot%202013-06-13%20at%203.01.46%20PM.png)


## Supported Languages and Services

Wercker currently supports Node.js, Python and Ruby. In terms of services like databases and queues, wercker has support for Postgres, MySQL, MongoDB, RabbitMQ, and Redis. See the [services section on our dev center](http://devcenter.wercker.com/articles/services/) on how to specify any of these.

## Add a badge

You can attach your build status through the wercker badge; for instance in the `README` of your repository on GitHub.

![image](http://f.cl.ly/items/0A1i3y2c3J1E2R0r2F0i/Screen%20Shot%202013-06-13%20at%202.45.46%20PM.png)

Again you can sign up for wercker [here](https://app.wercker.com/users/new/)

## Earn some stickers!

Let us know about the applications you build with Heroku and wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

