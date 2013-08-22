---
title: Simplifying SSH-based deployment with wercker
date: 2013-08-22
tags: deployment, ssh
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
published: false
---

<h4 class="subheader">
</h4>

This article is also available on our [devcenter]()

READMORE

For some wercker has had the capability to deploy to Platform-as-a-Service providers such as [Heroku](http://devcenter.wercker.com/articles/deployment/heroku.html) and [OpenShift](http://devcenter.wercker.com/articles/deployment/openshift.html). By using [pipeline steps]() you we're even able to deploy, for instance static sites built with [Jekyll](http://blog.wercker.com/2013/05/31/simplify-you-jekyll-publishing-process-with-wercker.html) to Amazon S3, using the [s3sync step](https://app.wercker.com/#applications/51c82a063179be4478002245/tab/details).

Through [scripting](http://devcenter.wercker.com/articles/deployment/capistrano.html) you we're most certainly able to deploy to Infrastructure-as-a-Service providers, this is how wercker itself is deployed, to [Amazon EC2](http://aws.amazon.com/ec2/).

Today we're happy to announce we've simplified the deployment process to IaaS provider such as [Digital Ocean](http://digitalocean.com), [Rackspace](http://www.rackspace.com/), EC2 and even [baremetal](http://www.hetzner.de/en/).

## How does it work

You are now able to generate an SSH key on wercker that you can leverage for deployment. After generating a key, you make it available as an environment variable, [12 Factor style](http://12factor.net) and leverage it in your deployment [pipeline](http://devcenter.wercker.com/articles/introduction/pipeline.html).

Within the **settings tab** of your application there is now a new
section called **Key management**, here you can generate a key pair
(public and private). The private key will only be available from within
either the deploy or build process, and is not displayed.

![image() #generate

After you have generated a key pair, you can now hook it up to a [deploy
target](http://devcenter.wercker.com/articles/introduction/deploys.html#deploy-targets).

![image]() # env var -> key pair

Upon creating a deploy target you are able to add either a text-based
environment variable, or couple or newly generated key pair to an
environment variable. The latter is what you want for deployment.

From the pull-down menu you are able to select your key pair. After
giving your environment variable a name, let's say for instance **FOO**, both the public and private key
will become available in the deployment pipeline as **FOO_PUBLIC** and
**FOO_PRIVATE**.

Let's put our knowledge of SSH-based deployment to work and deploy an
application from wercker to [Digital Ocean]().

## Deploying to Digital Ocean

This tutorial assumes you've created a droplet on [Digital
Ocean](http://digitalocean.com) and have
configured it with
[passwordless-login](https://www.digitalocean.com/community/articles/how-to-set-up-ssh-keys--2), as we'll, of course, be using SSH
keys!

The application we will be deploying is a simple golang application that
returns city names as JSON.
You can find the source code for it on [GitHub]().

First, we'll create a custom deploy target for our application. Let's
call the deploy target **digitalocean**. Here we also auto-deploy the
master branch of our application.

In the SSH menu for you droplet add the public key that you've just
created on wercker as explained in the previous section *How does it
work*.

![image](http://f.cl.ly/items/0o140V3V34120t430I0C/Screen%20Shot%202013-08-22%20at%202.46.54%20PM.png)

