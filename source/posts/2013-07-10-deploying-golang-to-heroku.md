# Deploying Golang to Heroku

In this post we will show you how you can build, test and deploy an golang application to heroku.

## Prerequisites

* You have [registered](https://id.heroku.com/signup) at heroku
* You have [registered](https://app.wercker.com/users/new/) at wercker
* You have the [heroku toolbelt](https://toolbelt.heroku.com/) installed

## Installing wercker cli

Heroku’s Adam Wiggins said it best: “Web UIs are great for many things, but command-line interfaces are the heart of developer workflows”. Wercker comes with an powerful CLI that enables you to builds and deploy your software without leaving your terminal windows.

    pip install wercker

Note: you can read the [Wercker CLI Installation](http://devcenter.wercker.com/articles/cli/installation.html)page on our devcenter for more details.

## Adding your Go project to heroku

If your application is already added to heroku and is successfully running, you can skip this step.

Heroku needs to know where your application should be placed in the Go directory hierarchy. You need to specify this with in a `.godir` file in the root of your repository that contains this path. For my [go-cities](https://github.com/pjvds/go-cities) application this needs to be `github.com/pjvds/go-cities`.

    $ echo "github.com/pjvds/go-cities" > .godir
    $ git add .godir
    $ git commit -m 'Adds .godir that specifies the root directory'

Heroku also needs to know how to start your application. This needs to be specified in an `Procfile`.

    $ echo "web ./go-cities -port $PORT" > Procfile
    $ git add Procfile
    $ git commit -m 'Adds Procfile to tell heroku how to run'

Now you can create an application at heroku that will host the Go application. Use the [Golang BuildPack] to add Go support to this application.

    $ heroku create -b https://github.com/kr/heroku-buildpack-go.git
    Creating ancient-temple-243... done, stack is cedar
    Git remote heroku added

## A note about how wercker builds software

In short, this is how wercker builds your software:

* Get the code by cloning the git repository and checking out the correct commit.
* Start an sandboxed environment that represents your stack of choice.
* Execute the build pipeline within this sandboxed environment.

This sandboxed environment is a set of virtual machines that wercker starts. These virtual machines are called boxes. There is at least one box, that box is used to execute the build pipeline. It should contain all the software that the build pipeline depends on. Wercker offers predefined boxes for common development stacks like Ruby, Python and NodeJS. But wercker also allows you to create your own box. Something I [have done for go](https://github.com/pjvds/box-golang).

## Adding an wercker.yml

For this project you want to use my golang box. You can tell this to wercker by adding an `wercker.yml` file with set the `box` element to `pjvds/golang`.

    $ echo "box: pjvds/golang" > wercker.yml
    $ git add wercker.yml
    $ git commit -m 'Adds wercker.yml'
    $ git push origin master

The golang box has - like most boxes at wercker - sensible defaults. The default for the go build pipeline is to execute the following command:

* Setup Go environment
* Execute `go get ./..` command to get dependencies
* Execute `go build ./..` command to build your software
* Execute `go test ./..` command to run your tests

## Add your project to wercker

Now it is time to add your application to wercker. You can leverage wercker addon for heroku to do most of the setup for you.

    $ heroku addons:add wercker
    Adding wercker on ancient-temple-243 done, v2 (free)
    Use `heroku addons:open wercker` to get started.
    Use `heroku addons:docs wercker` to view documentation.

By adding the addon you share the heroku application details with wercker. Wercker will use this information to create an deployment target for you.

Now you can actually create your can open the wercker addon page followed by a `wercker create` command to add add your application to wercker.

    $ wercker create

Wercker should now be building and testing your go code. You can use the `wercker status` command to get the status of the last builds on your project.

    $ wercker status
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

## Triggering an deployment to heroku

When your build passed successfully you can deploy it to heroku by executing the `wercker deploy` command. This command will list your last successful builds and lets you pick the target to deploy to. This is especially helpful when you have multiple targets like an staging and production environment.

    $ wercker deploy
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

Whenever you push new commits to your remote git repository wercker will start a new build. You can go to your application page at [app.wercker.com](https://app.wercker.com) to see the status of these builds or use the `wercker status` command from the CLI as demonstrated in this article. The `wercker deploy` command lets you deploy green builds to heroku.

You can also enable auto deploymet for specific brances in the settings tab of your application at [app.wercker.com](https://app.wercker.com). With this feature all green builds from an specific branch get deployed to heroku.

## Earn stickers!

Tweet out an screenshot of your successful build or deployment and I will make sure you receive some stickers.
