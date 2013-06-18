---
title: Spotlight on the wercker CLI
date: 2013-06-18
tags: wercker, cli
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
<a href="http://heroku.com">Heroku's</a> <a href="http://twitter.com/hirodusk">Adam
Wiggins</a> said it best: "Web UIs are great for many things,
but command-line interfaces are the heart of developer workflows". In
this post we will go into how the wercker command line interface works
and showcase some of wercker's terminal power.
</h4>

READMORE

## Introduction

The [wercker command line
interface](http://devcenter.wercker.com/articles/cli/) is written in Python and thus easily installable across various platforms, whether you're running on Mac, Linux or Windows.

We've open [sourced it on
GitHub](https://github.com/wercker/wercker-cli), so feel free to make contributions or
even port it to another programming language. You can find the CLI [on
wercker](https://app.wercker.com/#project/5162d11aa80dbabf410031e3) with
its latest build status. We deploy the CLI to [PyPI](http://pypi.python.org), the Python package
index and have written an indepth article on how we've structured our
deployment pipeline to PyPI on our [dev
center](http://devcenter.wercker.com/articles/deployment/pypi.html).

You can install the wercker command line interface as follows:

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

For your convenience we've already created an application in Node.js using
the [Express](http://expressjs.com/)  framework. Take a look on the
[wercker GitHub page](https://github.com/wercker/getting-started-nodejs) and go ahead and fork + clone
this application:

``` bash
git clone https://github.com/<YOUR_USERNAME>/getting-started-nodejs.git
```

Upon inspecting the project folder you will see that this is a simple
application that returns a list of cities as JSON. It also contains a unit test written with
[SuperTest](https://github.com/visionmedia/supertest).

There is also a `Procfile` present which we will use to deploy
our application to Heroku. In order to deploy to Heroku we need to create an application on Heroku first:

``` bash
heroku create
```

As we are going to deploy our application to Heroku we can install the [wercker add-on](https://addons.heroku.com/wercker), to make deployment even smoother.
The Heroku add-on is currently in [beta](http://blog.wercker.com/2013/06/13/Wercker-heroku-addon-in-beta.html) so make sure you are part of the [Heroku Beta Program](http://beta.heroku.com), before adding the add-on as follows:

``` bash
heroku addons:add wercker
```

We are now all set to add our application to wercker with the CLI!

### Adding an application to wercker

The first thing you want to do after installing the CLI is log into wercker
with your username and password:

``` bash
wercker login
```

Adding your repository to wercker is done by running:

``` bash
wercker create
```

Wercker will automatically check the [git
remotes](http://git-scm.com/book/en/Git-Basics-Working-with-Remotes) in your `.git`
folder and use that information to add your project to wercker. (the CLI
will also check if it can find an [Heroku
remote](https://devcenter.heroku.com/articles/git)).

Finally, the CLI will check if werckerbot is added as a collaborator to
GitHub or Bitbucket.

Upon running `wercker create` you will see the following output:

``` bash
-----------------------
welcome to wercker-cli
-----------------------

About to create an application on wercker.

This consists of the following steps:
1. Validate permissions and create an application
2. Add a deploy target (1 heroku targets detected)
3. Trigger initial build
```
The CLI will detect both your `git remote` for your repository on GitHub and the Heroku
remote that you want to deploy to. It will also check to see if `werckerbot` has access to your GitHub repository.
This is necessary to execute builds on the wercker platform. For more info on `werckerbot`
see the [article](http://devcenter.wercker.com/articles/gettingstarted/werckerbot.html) on
our [dev center](http://devcenter.wercker.com/). You will see the following output:

``` bash
Step 1.
-------------

Found 1 repository location(s)...

Please choose one of the following options:
 (1) git@github.com:mies/getting-started-nodejs.git
Make your choice (1=default):

github repository detected...
Selected repository url is git@github.com:mies/getting-started-nodejs.git

Creating a new application
a new application has been created.
In the root of this repository a .wercker file has been created which enables the link between the source code and wercker.

Checking werckerbot permissions on the repository...
Werckerbot has access
```

As said, the CLI will also detect your [Heroku git remote](https://devcenter.heroku.com/articles/git), making deployment even easier!
The CLI will tell you this as well and subsequently trigger a new build:

``` bash
Step 2.
-------------

1 automatic supported target(s) found.
Heroku remote git@heroku.com:vast-mountain-8769.git selected.
Heroku deploy target vast-mountain-8769 successfully added to the wercker application


Step 3.
-------------

Triggering build
A new build has been created

Done.
-------------

You are all set up to for using wercker. You can trigger new builds by
committing and pushing your latest changes.
```

### Deploying to Heroku

Before deploying our application, we of course want to check if our build has passed or not!
We do this by running:

``` bash
wercker status
```

This command retrieves the **latest** build status and presents the following output:

``` bash
-----------------------
welcome to wercker-cli
-----------------------

Retreiving builds from wercker...
Found 1 result(s)...

┌────────┬──────────┬────────┬──────────┬───────────────────┬─────────────┐
│ result │ progress │ branch │ hash     │ created           │ message     │
├────────┼──────────┼────────┼──────────┼───────────────────┼─────────────┤
│ passed │ 100.0%   │ master │ 9ea0deb1 │ 06/18/13 09:34:38 │ fixing test │
├────────┼──────────┼────────┼──────────┼───────────────────┼─────────────┤
```

If you want an overview of **all** your builds you run:

``` bash
wercker builds
```

That returns the following table:

``` bash
-----------------------
welcome to wercker-cli
-----------------------

Retreiving builds from wercker...
Found 2 result(s)...

┌────────┬──────────┬────────┬──────────┬───────────────────┬─────────────────┐
│ result │ progress │ branch │ hash     │ created           │ message         │
├────────┼──────────┼────────┼──────────┼───────────────────┼─────────────────┤
│ passed │ 100.0%   │ master │ 9ea0deb1 │ 06/18/13 09:34:38 │ fixing test     │
├────────┼──────────┼────────┼──────────┼───────────────────┼─────────────────┤
│ failed │    -     │ master │ c19dba5c │ 06/18/13 09:17:08 │ added gitignore │
├────────┼──────────┼────────┼──────────┼───────────────────┼─────────────────┤
```

As you can see from the previous table, our latest build has passed and we are ready to deploy our app with the following command:

``` bash
wercker deploy
```
This will present a small interactive menu where you can select the **passed** build to deploy (we have only one!),
and to which deploy target you want to deploy (we only created one for Heroku). In both cases the CLI defaults to the most sensible option (latest passed build and our only deploy target)

``` bash
-----------------------
welcome to wercker-cli
-----------------------

Retreiving builds from wercker...
Found 1 result(s)...

┌───┬────────┬──────────┬────────┬──────────┬───────────────────┬─────────────┐
│   │ result │ progress │ branch │ hash     │ created           │ message     │
├───┼────────┼──────────┼────────┼──────────┼───────────────────┼─────────────┤
│ 1 │ passed │ 100.0%   │ master │ 9ea0deb1 │ 06/18/13 09:34:38 │ fixing test │
├───┼────────┼──────────┼────────┼──────────┼───────────────────┼─────────────┤
Select which build to deploy(enter=1):

Retreiving list of deploy targets...
Found 1 result(s)...

┌───┬────────────────────┬────────┬───────────┬─────────────┬────────┬────────┬─────────┐
│   │ target             │ result │ deploy by │ deployed on │ branch │ commit │ message │
├───┼────────────────────┼────────┼───────────┼─────────────┼────────┼────────┼─────────┤
│ 1 │ vast-mountain-8769 │ -      │ -         │ -           │ -      │ -      │ -       │
├───┼────────────────────┼────────┼───────────┼─────────────┼────────┼────────┼─────────┤
Select a target to deploy to(enter=1):
Success:
            Build scheduled for deploy.

You can monitor the scheduled deploy in your browser using:
wercker targets deploy
Or query the queue for this application using:
wercker queue
```

### Other useful commands

As the above output indicates, we can run the command `wercker queue` to monitor the progress
of our builds and deploys.

We can also check if we have the latest version of the CLI by running

``` bash
wercker update
```

Finally, an overview of all your apps on wercker and their latest build status can be achieved through:

``` bash
wercker apps
```

This will present the following table:

``` bash
├──────────────────────────────────────┼────────────────┼─────────┼───────────┼───────────────────────────────────────────────────────────────────┤
│ getting-started-nodejs               │ mies           │ passed  │ 2         │ git@github.com:wercker/getting-started-nodejs.git                 │
| middleman                            │ mies           │ passed  │ 1         │ git@github.com:mies/middleman.git                                 │
│ middleman-guides                     │ mies           │ passed  │ 1         │ git@github.com:mies/middleman-guides.git                          │
│ node-openshiftclient                 │ wercker        │ passed  │ 7         │ git@github.com:wercker/node-openshiftclient.git                   │
│ yoga                                 │ zeke           │ failed  │ 3         │ git@github.com:zeke/yoga.git                                      │
├──────────────────────────────────────┼────────────────┼─────────┼───────────┼───────────────────────────────────────────────────────────────────┤
```

The CLI offers a powerful way of interacting with wercker from the comfort of your terminal!

Let us know what you think of the CLI and don't forget to [sign up for wercker](https://app.wercker.com/users/new/)!

