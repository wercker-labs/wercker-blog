---
title: Simplifying SSH-based deployment with wercker
date: 2013-08-22
tags: deployment, ssh
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
    In this article we will dive into how to do continous delivery from wercker to cloud providers such as AWS, RackSpace and Digital Ocean.
</h4>

![image](http://f.cl.ly/items/233z3o2D0M2D0H2L0s3Y/wercker%2Bsshkeys.png)

READMORE

For some time wercker has had the capability to deploy to Platform-as-a-Service providers such as [Heroku](http://devcenter.wercker.com/articles/deployment/heroku.html) and [OpenShift](http://devcenter.wercker.com/articles/deployment/openshift.html). By using [pipeline steps]() you we're even able to deploy, for instance static sites built with [Jekyll](http://blog.wercker.com/2013/05/31/simplify-you-jekyll-publishing-process-with-wercker.html) to Amazon S3, using the [s3sync step](https://app.wercker.com/#applications/51c82a063179be4478002245/tab/details).

Through [scripting](http://devcenter.wercker.com/articles/deployment/capistrano.html) you we're most certainly able to deploy to Infrastructure-as-a-Service providers, this is how wercker itself is deployed, to [Amazon EC2](http://aws.amazon.com/ec2/).

Today we're happy to announce we've simplified the deployment process to IaaS provider such as [Digital Ocean](http://digitalocean.com), [Rackspace](http://www.rackspace.com/), EC2 and even [baremetal](http://www.hetzner.de/en/).

## How does it work

You are now able to generate an SSH key on wercker that you can leverage for deployment. After generating a key, you make it available as an environment variable, [12 Factor style](http://12factor.net) and leverage it in your deployment [pipeline](http://devcenter.wercker.com/articles/introduction/pipeline.html).

Within the **settings tab** of your application there is now a new
section called **Key management**, here you can generate a key pair
(public and private). The private key will only be available from within
either the deploy or build process, and is not displayed.

![image](http://f.cl.ly/items/3x0P3u1X3P2Z1E3A2Z2t/Screen%20Shot%202013-08-23%20at%202.50.10%20PM.png)

After you have generated a key pair, you can now hook it up to a [deploy
target](http://devcenter.wercker.com/articles/introduction/deploys.html#deploy-targets).

![image](http://f.cl.ly/items/0V2e2h0i0j1G2b1V161N/Screen%20Shot%202013-08-23%20at%202.52.02%20PM.png)

Upon creating a deploy target you are able to add either a text-based
environment variable, or couple or newly generated key pair to an
environment variable. The latter is what you want for deployment.

From the pull-down menu you are able to select your key pair. After
giving your environment variable a name, let's say for instance **FOO**, both the public and private key
will become available in the deployment pipeline as **FOO_PUBLIC** and
**FOO_PRIVATE**.

Let's put our knowledge of SSH-based deployment to work and deploy an
application from wercker to [Digital Ocean](http://digitalocean.com).

## Deploying to Digital Ocean

This tutorial assumes you've created a droplet on [Digital
Ocean](http://digitalocean.com) and have
configured it with
[passwordless-login](https://www.digitalocean.com/community/articles/how-to-set-up-ssh-keys--2), as we'll, of course, be using SSH
keys! If this is a new droplet make sure you first add the SSH key to
Digital Ocean using their dashboard before spinning up the server. If
this is an existing droplet you will want to add the previously created
key to the **authorized_keys** on your droplet.
See
Digital Ocean's SSH
[article](https://www.digitalocean.com/community/articles/how-to-use-ssh-keys-with-digitalocean-droplets)
for more information.

The application we will be deploying is a simple golang application that
returns city names as JSON.
You can find the source code for it on [GitHub](https://github.com/mies/getting-started-golang/).

First, we'll create a custom deploy target for our application. Let's
call the deploy target **digitalocean**. Here we also auto-deploy the
master branch of our application.

![image](http://f.cl.ly/items/2S2s3f441A1Z2o3E2i31/Screen%20Shot%202013-08-22%20at%203.01.36%20PM.png)

Now follow the instructions from the previous section *How does it
work?*, to generate an SSH key pair and add it as an environment
variable to your deploy target.

![image](http://f.cl.ly/items/0o140V3V34120t430I0C/Screen%20Shot%202013-08-22%20at%202.46.54%20PM.png)

Again, make sure you add this public key to your authorized keys on your
droplet.

We need a way to start deployed applications on our Digital Ocean server. I'm using ubuntu so upstart is available to me and will use it to stop and start my application.
You can read more about upstart [here](http://upstart.ubuntu.com).

### Creating our wercker.yml

The [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file is the way to declare both your build and deployment pipeline on wercker. Below you the `wercker.yml` is presented for this application. The build section fetches dependencies for our app, compiles our golang code and runs the unittests. As golang requires a separate [workspace environment](http://golang.org/doc/code.html), which is set up through the [setup-go-workspace](https://app.wercker.com/#applications/51fa5e6ba4037f7171000f75/tab/details) step, we need to copy our source folder to the `$WERCKER_OUTPUT_DIR` containing the binary needed for deployment.

``` yaml
box: wercker/golang
# Build definition
build:
  # The steps that will be executed on build
  steps:
    # Sets the go workspace and places you package
    # at the right place in the workspace tree
    - setup-go-workspace

    # Gets the dependencies
    - script:
        name: go get
        code: |
          cd $WERCKER_SOURCE_DIR
          go version
          go get ./...

    # Build the project
    - script:
        name: go build
        code: |
          go build ./...

    # Test the project
    - script:
        name: go test
        code: |
          go test ./...
    - script:
        name: copy output
        code: |-
          rsync -avz "$WERCKER_SOURCE_DIR/" "$WERCKER_OUTPUT_DIR"
```
The deploy section is where it gets interesting! Here we leverage various [steps](http://devcenter.wercker.com/articles/steps/) available in the [wercker directory](https://app.wercker.com/#explore).

``` yaml
deploy:
  steps:
  - add-to-known_hosts:
        hostname: YOURSERVER.COM
   - mktemp:
      envvar: PRIVATEKEY_PATH
   - create-file:
      name: write key
      filename: $PRIVATEKEY_PATH
      content: $FOO_PRIVATE
      overwrite: true
      hide-from-log: true
   - script:
      name: stop application
      code: ssh -i $PRIVATEKEY_PATH -l root -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no YOURSERVER.COM stop gocities
   - script:
      name: transfer application
      code: |
        pwd
        ls -la
        scp -i $PRIVATEKEY_PATH -o StrictHostKeyChecking=no -o UserKnownHostsFile=no digitalocean-test root@YOURSERVER.COM:/var/local/www
   - script:
      name: start application
      code: ssh -i $PRIVATEKEY_PATH -l root -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no YOURSERVER.COM start gocities
```

First, we need to add the server we want to deploy to, to the known hosts file. We can do this using the [add-to-known_hosts step](https://app.wercker.com/#applications/521764dde36a64ff110022f2/tab/details).

Second, we create a temporary file and make it available as an environment variable using the [mktemp step](https://app.wercker.com/#applications/52167277e9fa619606001064/tab/details). This filepointer will contain our private key that we've previously created.

Next, we write the contents of our private key (`$FOO_PRIVATE`) to the temporary file we created in the previous step using `mktemp`. We use the [create-file step](https://app.wercker.com/#applications/51c829dd3179be4478002113/tab/details) to do so. Now we are ready to communicate with our server!

As, I've already got my application running on my host, I need to stop it first, thus I leverage a custom `script` step that stops my app, configured as `gocities` through **upstart**. This could vary for your own setup of course, but should put you on the right track.

Next, I transfer my application, the compiled binary, using scp to my remote `/var/local/www/` folder.

As a final step, I start my application, again through **upstart**.

If you add this `wercker.yml` file to your git repostory and push it, the build will deploy to the droplet automatically (if you have auto-deploy enabled, otherwise you would have to trigger it manually).

Our application is now running on my Digital Ocean droplet, delivered by wercker!

![image](http://f.cl.ly/items/2q3c2J1a3H0N1L1E2715/Screen%20Shot%202013-08-23%20at%204.50.25%20PM.png)

We will discuss SSH-based deployment to other cloud providers in future posts as well!

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.
