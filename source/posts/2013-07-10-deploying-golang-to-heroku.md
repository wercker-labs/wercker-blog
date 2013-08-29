---
title: Build, test and deploy a Go application to Heroku
date: 2013-07-10
tags: opendelivery, boxes, golang, heroku
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
---

<h4 class="subheader">
In a previous post we briefly outlined how to get started with <a href="http://blog.wercker.com/2013/07/10/Golang-on-wercker.html">golang</a> and wercker. Now we are going to do a deepdive into golang, wercker and Heroku.
</h4>

![image](https://s3.amazonaws.com/uploads.hipchat.com/36172/251802/hyi5dw0l78894x1/upload.png)

In this post I will share how you can build, test and deploy a golang application to [Heroku](http://heroku.com) and the Heroku wercker [add-on](https://addons.heroku.com/wercker).

## Prerequisites

* You have [registered](https://id.heroku.com/signup) for a Heroku account
* You have [registered](https://app.wercker.com/users/new/) for a wercker account
* You have the [heroku toolbelt](https://toolbelt.heroku.com/) installed

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/). We've open sourced the code for this application on [GitHub](https://github.com/pjvds/go-cities).

## Installing the wercker cli

Heroku’s Adam Wiggins said it best: "Web UIs are great for many things, but command-line interfaces are the heart of developer workflows". Wercker comes with a powerful CLI that enables you to build and deploy your software without leaving your terminal.

    pip install wercker

Note: you can read the [Wercker CLI Installation](http://devcenter.wercker.com/articles/cli/installation.html) page on our devcenter for more details.

## Adding your Go project to Heroku

If your application is already added to Heroku, you can skip this step.

Heroku needs to know where your application should be placed in the Go directory hierarchy. You need to specify this with a `.godir` file in the root of your repository that contains this path. For my [go-cities](https://github.com/pjvds/go-cities) application this needs to be `github.com/pjvds/go-cities`.

    echo "github.com/pjvds/go-cities" > .godir
    git add .godir
    git commit -m 'Adds .godir that specifies the root directory'

Heroku also needs to know how to start your application. This needs to be specified using a `Procfile`.

    echo "web: ./go-cities -port $PORT" > Procfile
    git add Procfile
    git commit -m 'Adds Procfile to tell heroku how to run'

Now you can create an application at Heroku that will host the Go application. Use the [golang](http://mmcgrana.github.io/2012/09/getting-started-with-go-on-heroku.html) [BuildPack](https://github.com/kr/heroku-buildpack-go) to add Go support for this application.

    heroku create -b https://github.com/kr/heroku-buildpack-go.git
    Creating ancient-temple-243... done, stack is cedar
    Git remote heroku added

## A note about how wercker builds software

Before you add your application to wercker it is important to understand the basics of how wercker builds and tests your software.

* Wercker builds your code on every `git push`
* It fetches the code by cloning the git repository and checking out the correct commit.
* A sandboxed environment is started that represents your stack of choice.
* The build pipeline is executed within this sandboxed environment.

This sandboxed environment is a set of virtual machines that wercker starts. These virtual machines are called boxes. There is at least one box, the box that is used to execute the build pipeline. It should contain all the software that the build pipeline depends on. Wercker offers predefined boxes for common development stacks like Ruby, Python and NodeJS. But wercker also allows you to create your own box. Something I [have done for go](https://github.com/pjvds/box-golang).

## Adding an wercker.yml

For this project you want to use my golang box. You can wercker know by adding a `wercker.yml` file with the `box` element set to `pjvds/golang`.

    echo "box: pjvds/golang" > wercker.yml
    git add wercker.yml
    git commit -m 'Adds wercker.yml'
    git push origin master

The golang box has - like most boxes on wercker - sensible defaults. These defaults for the go build pipeline are the following:

* Setup Go environment
* Execute `go get ./..` command to get dependencies
* Execute `go build ./..` command to build your software
* Execute `go test ./..` command to run your tests

## Add your project to wercker

Now it's time to add your application to wercker. You can the leverage [wercker addon](https://addons.heroku.com/wercker) which we've [recently released](http://blog.wercker.com/2013/07/10/Heroku-addon-public-beta.html), for Heroku to do most of the setup for you.

    heroku addons:add wercker
    Adding wercker on ancient-temple-243 done, v2 (free)
    Use `heroku addons:open wercker` to get started.
    Use `heroku addons:docs wercker` to view documentation.

By provisioning the add-on wercker automatically creates a deploy target to Heroku for you.

Now you can actually open the wercker add-on page, `heroku addons:open wercker` followed by a `wercker create` command to add your application to wercker.

First open the wercker add-on page on Heroku:

``` bash
heroku addons:open wercker
```

and then create your app on wercker:

``` bash
wercker create
```

Wercker should now be building and testing your Go code. You can use the `wercker status` command to get the status of the last builds of your project.

    wercker status
    -----------------------
    welcome to wercker-cli
    -----------------------

    Retreiving builds from wercker...
    Found 1 result(s)...

    ┌────────┬──────────┬────────┬──────────┬───────────────────┬─────────────────────────────────────────┐
    │ result │ progress │ branch │ hash     │ created           │ message                                 │
    ├────────┼──────────┼────────┼──────────┼───────────────────┼─────────────────────────────────────────┤
    │ passed │ 100.0%   │ master │ 0687f5fd │ 07/10/13 13:04:05 │ Adds Procfile to tell heroku how to run │
    ├────────┼──────────┼────────┼──────────┼───────────────────┼─────────────────────────────────────────┤

## Triggering a deploy to heroku

When your build passed successfully you can deploy it to Heroku by executing the `wercker deploy` command. This command will list your last successful builds and lets you pick the target to deploy to. This is especially helpful when you have multiple targets such as a staging and production environment.

    wercker deploy
    -----------------------
    welcome to wercker-cli
    -----------------------

    Retreiving builds from wercker...
    Found 2 result(s)...

    ┌───┬────────┬──────────┬────────┬──────────┬───────────────────┬─────────────────────────────────────────┐
    │   │ result │ progress │ branch │ hash     │ created           │ message                                 │
    ├───┼────────┼──────────┼────────┼──────────┼───────────────────┼─────────────────────────────────────────┤
    │ 1 │ passed │ 100.0%   │ master │ 0687f5fd │ 07/10/13 13:04:05 │ Adds Procfile to tell heroku how to run │
    ├───┼────────┼──────────┼────────┼──────────┼───────────────────┼─────────────────────────────────────────┤
    │ 2 │ passed │ 100.0%   │ master │ 5cc5d03c │ 07/10/13 12:44:42 │ Remove steps from wercker.yml           │
    ├───┼────────┼──────────┼────────┼──────────┼───────────────────┼─────────────────────────────────────────┤
    Select which build to deploy(enter=1): 1

    Retreiving list of deploy targets...
    Found 1 result(s)...

    ┌───┬────────────────────────┬────────┬───────────┬───────────────────┬────────┬──────────┬─────────────────────────────────────────┐
    │   │ target                 │ result │ deploy by │ deployed on       │ branch │ commit   │ message                                 │
    ├───┼────────────────────────┼────────┼───────────┼───────────────────┼────────┼──────────┼─────────────────────────────────────────┤
    │ 1 │ mysterious-sierra-9493 │ passed │ pjvds     │ 07/10/13 13:10:34 │ master │ 0687f5fd │ Adds Procfile to tell heroku how to run │
    ├───┼────────────────────────┼────────┼───────────┼───────────────────┼────────┼──────────┼─────────────────────────────────────────┤
    Select a target to deploy to(enter=1): 1
    Success:
                Build scheduled for deploy.

    You can monitor the scheduled deploy in your browser using:
    wercker targets deploy
    Or query the queue for this application using:
    wercker queue

## Continue

Whenever you push new commits to your remote git repository wercker will start a new build. You can go to your application page at [app.wercker.com](https://app.wercker.com) to see the status of these builds or use the `wercker status` command from the CLI as demonstrated in this article. The `wercker deploy` command lets you deploy green builds to Heroku.

You can also enable auto deploymet for specific brances in the settings tab of your application at [app.wercker.com](https://app.wercker.com). With this feature all green builds from an specific branch get deployed to Heroku.

## Earn stickers!

Tweet out a screenshot of your successful build or deployment with **#wercker** and I will make sure you receive some wercker stickers.
