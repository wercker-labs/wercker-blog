---
title: Spotlight on the wercker CLI
date: 2013-06-07
tags: wercker, cli
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
published: false
---

<h4 class="subheader">
[Heroku's](http://heroku.com) [Adam
Wiggins](http://twitter.com/hirodusk) said it best: "Web UIs are great for many things,
but command-line interfaces are the heart of developer workflows". In
this post we will go into how the wercker command line interface works
and showcase some wercker's terminal power.
</h4>

READMORE

## Introduction

The [wercker command line
interface](http://devcenter.wercker.com/articles/cli/) is written in Python and thus easily installable across various platforms, whether your running on Mac, Linux or Windows. You can
install it as follows:

``` bash
pip install python
```

Executing the `wercker` command will give the following overview of the available commands:

``` bash
$ wercker

-----------------------
welcome to wercker-cli
-----------------------

Usage:
    wercker create
    wercker status
    wercker deploy
    wercker builds
    wercker open targets
    wercker queue
    wercker apps
    wercker link
    wercker login
    wercker logout
    wercker targets add
    wercker targets list
    wercker targets details
    wercker validate
    wercker update
    wercker --version
    wercker --help
```

In the next paragraphs we will give an overview of the most important
commands. But first we need a small application that we want to add to
wercker and make actual use of the commandline! 

For your convenience we've already created an application in Ruby using
the [sinatra](http://sinatrarb.com) micro framework. Take a look on the
[werkcer GitHub page](https://github.com/wercker/getting-started-ruby) and go ahead and clone
this application:

``` bash
git clone https://github.com/wercker/getting-started-ruby.git
```

Go into your project folder and you will see, that indeed it is a simple
sinatra API that returns a collection of city names in JSON. It also has
a unit test.

The repository also contains a `Procfile` which we will use to deploy
our application to Heroku


### Adding an application to wercker

wercker login

wercker create

wercker status

wercker builds

wercker deploy

wercker update

