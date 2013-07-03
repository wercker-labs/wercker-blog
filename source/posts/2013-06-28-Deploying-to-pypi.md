---
title: Deploying python packages to PyPI with wercker
date: 2013-06-28
tags: wercker, python, pypi, deployment
author: Jacco Flenter
gravatarhash: 7d9ef3d3f6911e6e4f9c51f6d99c48f8
---

<h4 class="subheader">
Wercker allows you to deploy your application to various Platform-as-a-Service (PaaS) providers such as <a href="http://blog.wercker.com/2013/06/13/Wercker-heroku-addon-in-beta.html">Heroku</a> but also Infrastructure-as-a-Service providers including Amazon Web Services and Rackspace. We call these deploy targets.
We view deployment as a broad topic though, as we've previously showcased with our <a href="http://blog.wercker.com/2013/06/10/Streamlining-Middleman-Deploys-to-s3.html">posts</a> on <a href="http://blog.wercker.com/2013/05/31/simplify-you-jekyll-publishing-process-with-wercker.html">deploying</a> static sites built with <a href="http://middlemanapp.com">Middleman</a> or <a href="http://jekyllrb.com/">Jekyll</a> to S3. In this post
we'll be discussing how to deploy your Python packages to <a href="http://pypi.python.org/pypi">PyPI</a>, the Python package index.
</h4>

READMORE

## The wercker command line interface

As always, signing up for wercker is [free and easy](https://app.wercker.com/users/new/). This post is also available in short-form on our [dev center](http://devcenter.wercker.com/articles/deployment/pypi.html).

The [package](https://pypi.python.org/pypi/wercker/0.7.1) that we will be deploying to PyPI is the [wercker command line interface](http://devcenter.wercker.com/articles/cli/).

The wercker CLI is written in Python. Briefly, you can install this CLI via the `pip` package manager:

    $ pip install wercker

We've previously written about the command line interface and its possibilities in a post [here](http://blog.wercker.com/2013/06/18/Spotlight-on-the-wercker-cli.html).

Pip fetches the wercker CLI from [PyPI](https://pypi.python.org/), the Python package repository. Each time we update the CLI we want to be able to deploy it to the PyPI index automatically.

![image](http://f.cl.ly/items/1P2q2p0P1q1T021R0j1i/Screen%20Shot%202013-06-28%20at%202.38.04%20PM.png)

You can imagine the same scenario for other libraries or projects that you want to deploy to platforms such as [RubyGems](http://rubygems.org/) or [NPM](http://npmjs.org).

## Prerequisites

For this tutorial we assumes a few things:

* you have an account on PyPI. You can register at [https://pypi.python.org/pypi?%3Aaction=register_form](https://pypi.python.org/pypi?%3Aaction=register_form)
* a tool/library you want to push to PyPI. More information on what you need for PyPI can be found in the [The Hitchhiker’s Guide to Packaging](http://guide.python-distribute.org/index.html)
* your application is already on wercker (i.e. you have already run `wercker create` or have added your application via the web interface). See [getting started with the wercker CLI](http://devcenter.wercker.com/articles/gettingstarted/cli.html) or adding your application via the [web interface](http://devcenter.wercker.com/articles/gettingstarted/web.html) for more information on adding your application to wercker.
* the package you want to deploy to PyPI has a passed, or *green*, build on [wercker](http://devcenter.wercker.com/articles/introduction/builds.html).

## Creating your wercker.yml file

Wercker manages any steps it needs to execute through a simple configuration file called `wercker.yml`. This file basically defines your build and deployment pipeline. For more information see the [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) devcenter article.

``` yaml
deploy:
    steps:
        - script:
            name: pypi deploy
            code: ./deploy.sh
```

We have defined a single directive, called `deploy` in this `wercker.yml` file. After a deploy is triggered, the `deploy.sh` script will be executed.

## Creating your deploy script.

We are now ready to create our actual deploy script that will publish the wercker CLI to PyPI. The method for submitting packages to is detailed on the PyPI [documentation site](http://docs.python.org/3/distutils/packageindex.html). Suffice it to say that the upload is executed through the `python setup.py sdist upload` command.

``` bash
echo "[server-login]" > ~/.pypirc
echo "username:" $PYPI_USER >> ~/.pypirc
echo "password:" $PYPI_PASSWORD >> ~/.pypirc
python setup.py sdist upload
```

What is more interesting in this deploy script is the usage of environment variables. The `python setup.py sdist upload` command expects a `.pyirc` file in your `$HOME` folder that contains your PyPI username and password. Within wercker you ara able to define these environment variables for a deploy target.

## Add a deploy target.

Go to [wercker](https://app.wercker.com) and add a custom deploy target to your application. Name it pypi and add the two environment varaibles Now there are two environment variables we want to add, PYPI\_USER and PYPI\_PASSWORD. We may want to check the "hidden from log" checkbox, since it is not relevant to see this information in our deploy log.

![image](http://f.cl.ly/items/0I2f3x2D1u3H1F3q0A2y/Screen%20Shot%202013-06-28%20at%202.32.50%20PM.png)

That's all, you can now deploy your green builds by running `wercker deploy` (or of course deploying from the web interface). You can see if your deploy has been succesful by going to the 'deploys' tab on wercker, as showcased below:

![image](http://f.cl.ly/items/3I0O1g1l2d1e252u0x2m/Screen%20Shot%202013-06-28%20at%202.34.19%20PM.png)

We can imagine this being useful for other types of repositories or packages indexes as well! Let us know what you cook up with wercker.

### Earn some stickers!

Tell us about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).