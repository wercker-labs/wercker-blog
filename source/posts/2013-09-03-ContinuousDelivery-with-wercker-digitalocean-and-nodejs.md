---
title: Continuous Delivery with wercker and Digital Ocean
date: 2013-09-03
tags: deployment, digitalocean, nodejs
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
In this article we are going to a deep dive on how to do continous delivery for your applications from wercker to Digital Ocean.
</h4>

![image](http://f.cl.ly/items/0p2t3D2b3A323O2j2Q1L/Image%202013.09.02%201%3A48%3A58%20PM.png)

You can sign up for wercker for free [here](https://app.wercker.com/users/new).

READMORE

For this tutorial we will be building a small API that returns the Latin names of various cloud formations in JSON. We will also create a unit test that goes along with our API in order to make sure it is functioning properly. Both the application and unittest will be written in [node.js](http://nodejs.org), a popular language for building web applications.

You can check out the source code of this application on [GitHub]()

## Creating our application
Let's start building our app! First, create a `git` repository that will hold our source code:

~~~~bash
mkdir digitalocean-wercker-nodejs
cd digitalocean-wercker-nodejs
git init
~~~~

Now, lets declare our dependencies through node.js' `package.json` file:

~~~~json
{
  "name": "digitalocean-wercker-nodejs",
  "version": "0.0.1",
  "engines" : {
  "node": "0.10.x"
  },
  "dependencies": {
    "express": "3.x",
    "supertest" : "0.4.0",
    "mocha" : "1.6.0"
  }
}
~~~~

Here we specify the details of our application, most importantly the dependencies. Our API leverages the [express.js framework](http://expressjs.com/). We also declare dependencies for **supertest** and **mocha**, which we will use for our unittest.

Next, we're ready to build our actual application. Create a filed called `app.js` with the following contents:

~~~~javascript
var express = require('express');
var app = express();

app.get('/', function(req, res){
  res.send("Hello from digital ocean and wercker!")
});

app.get('/clouds.json', function(req, res){
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.write(JSON.stringify({clous : ["Altocumulus ", "Altostratus", "Cumulonimbus", "Nimbostratus", "Cirrocumulus", "Stratus"]}));
  res.end();
});

var port = 80;
app.listen(port);

module.exports = app;
~~~~

Now push your code to your version control provider of choice. Wercker supports either [GitHub](http://github.com) or [Bitbucket](http://bitbucket.org).

~~~~bash
git add .
git commit -am 'initial commit'
git push origin master
~~~~

## Adding our application to wercker

Now, we're ready to add our application to wercker. Make sure you've [created an account](https://app.wercker.com/users/new) and then click the add application button from the sidebar on the left.

![image](http://f.cl.ly/items/410k3B3a1X2w1H0G3D0A/Screen%20Shot%202013-09-02%20at%201.04.20%20PM.png)

This will open up the add application wizard which will guide you through the necessary steps for setting up your app with wercker.

First, select your git provider that hosts the repository that you just pushed to, either GitHub or Bitbucket.

![image](http://f.cl.ly/items/1w3e3E1310103E1J0I3g/Screen%20Shot%202013-09-02%20at%2011.10.48%20AM.png)

Next, select your repository, in my case, I've named it **digitalocean-wercker-nodejs**.

![image](http://f.cl.ly/items/111Z0R0O0C0r3j3t0X07/Screen%20Shot%202013-09-02%20at%2011.11.34%20AM.png)

If your application is public you don't have to do anything in the next step, if it is a private repository you need to add the **werckerbot** user as a collabarator to your repository.

Below you can find a screenshot that showcases this for GitHub.

![image](http://f.cl.ly/items/0B2D2R341D2T11083b1c/Screen%20Shot%202013-09-02%20at%2011.12.17%20AM.png)

The next step, helps you with setting up your [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file. Wercker uses a small *domain specific language*, or DSL, to define your build and deployment environment on wercker. This file is written in the [YAML](http://yaml.org/) language.

Based on your source code, wercker will make suggestions for your **wercker.yml** file. As this is a **node.js** project, the suggested wercker.yml contains [steps](http://devcenter.wercker.com/articles/steps/) for installing the node.js dependencies that we've declared in our **package.json** and running any unit test (which we will add at a later stage!). Copy and paste the suggested contents into a file called `wercker.yml` in your project folder.

Now finish up the wizard, as a last step you can make your application public or private. Public projects are perfect for open source or small test projects, so others can view the build status. If it is a company project you can keep your project private.

![image](http://f.cl.ly/items/1z0337251W3Y2P1e430J/Screen%20Shot%202013-09-02%20at%2011.14.58%20AM.png)

Your project has now been added and you will be greeted by the wercker dashboard.

![image](http://f.cl.ly/items/0t0h3r2b2O3h2V1U1D0g/Screen%20Shot%202013-09-02%20at%2011.16.56%20AM.png)

You can trigger a build manually but lets add the **wercker.yml** file which we need to add to our repository anyway. Add this file to your project repository and push it to git. Changes to the wercker.yml that are pushed will be picked up by wercker automatically.

~~~~bash
git add wercker.yml
git commit -am 'added wercker.yml'
git push origin master
~~~~

As you will see, the `git push` triggers a new build that looks as follows:

![image](http://f.cl.ly/items/3q1U2Y2O0D31160X0W00/Screen%20Shot%202013-09-02%20at%2011.21.11%20AM.png)

## Creating a unittest

Let's also create a unittest for our application. Add a folder called `test` to your repository and in this folder add a file named `test.js`.

~~~~bash
mkdir test
touch test/test.js
~~~~
Add the following contents to the `test.js` file:

~~~~javascript
var request = require('supertest')
  , express = require('express');

var app = require('../app.js');

describe('GET', function(){
  it('respond with json', function(done){
    request(app)
    .get('/clouds.json')
    .set('Accept', 'application/json')
    .expect('Content-Type', /json/)
    .expect(200, done);
  })
})
~~~~

The unittest checks if our applications returns JSON at the url that we've defined in our `app.js` file.

We need to update our `package.json` file such that our test will be run on wercker when we run `npm test`. Make sure your `package.json` file looks as follows:

~~~~json
{
  "name": "digitalocean-wercker-nodejs",
  "version": "0.0.1",
  "engines" : {
  "node": "0.10.x"
  },
  "dependencies": {
    "express": "3.x",
    "supertest" : "0.4.0",
    "mocha" : "1.6.0"
  },
  "scripts": {
    "test": "mocha",
    "start": "app.js"
  }
}
~~~~

We've included a `scripts` section that runs the `mocha` test framework when `npm test ` called.

As the `wercker.yml` suggested by wercker already runs `npm test` by default we are good to go. Add and commit our unittest and updated `package.json` to your repository and push it.

~~~~bash
git add tests
git commit -am 'added unittest and updated package.json`
git push origin master
~~~~

This will trigger a new build on wercker which will also run our unittest. This build should be green, meaning we're ready for deployment!

## Setting up our deploy target

We now need a target to deploy to, which of course will be a Digital Ocean droplet! We will create our droplet at a later stage, but first we're going to setup the communication between wercker and our droplet, via SSH keys.

In the **Settings** tab of your application, go to the *Key management* section. Here you can generate a key pair (public and private). The private key will only be available from within either the deploy or build process, and is not displayed.

![image](http://f.cl.ly/items/3x0P3u1X3P2Z1E3A2Z2t/Screen%20Shot%202013-08-23%20at%202.50.10%20PM.png)

Make sure you copy the public key to your clipboard as we will use it later.

After you have generated a key pair (which I've named **foo**), you can now hook it up to a deploy target, which we'll create now.

In the same **Settings** tab where you just were, now go to the *Deploy target* section and click the *add deploy target* button, and select **Custom deploy target**. Give it a name such as "DigitalOcean" and navigate tot the **Deploy pipeline** subsection. Here we will couple our created keypair to an environment variable, such that the key is available in our deploy pipeline, thus allowing us to deploy to Digital Ocean.

![image](http://f.cl.ly/items/2P163u1h440J3d433k2K/Screen%20Shot%202013-09-02%20at%203.06.25%20PM.png)

Give the environment variable a name such as **WERCKER**. The public and private key will subsequently be exposed in the deployment pipeline as environment variables named **WERCKER_PUBLIC** and **WERCKER_PRIVATE**.

## Creating our Digital Ocean Droplet

Now, log into your Digital Ocean dashboard and go to the SSH section, hit the **Add SSH Key** button and paste the public key that you've previously created at wercker. Let's just call it **wercker**.

![image](http://f.cl.ly/items/1y251o312o332G1x0h3T/Screen%20Shot%202013-09-03%20at%2010.51.07%20AM.png)

When we create a new droplet this key will be added by default. If you want to use this key with an *existing* droplet you will want to add the previously created key to the **authorized_keys** on your droplet. See Digital Ocean's [SSH article](https://www.digitalocean.com/community/articles/how-to-use-ssh-keys-with-digitalocean-droplets) for more information. This tutorial assumes you've created a droplet on Digital Ocean and have configured it with passwordless-login, see the [Digital Ocean article](https://www.digitalocean.com/community/articles/how-to-set-up-ssh-keys--2) for more information to configure your droplet as such.

Let's provision our droplet so we're able to run nodejs applications. First, install some prerequisites:

~~~~bash
sudo apt-get install python-software-properties python g++ make
~~~~

If you're droplet is running Ubuntu 12.10 you will need to install the following package as well:

~~~~bash
sudo apt-get install software-properties-common
~~~~
Add the node.js PPA as recommended by the official maintainers of node.js, Joyent.

~~~~bash
sudo add-apt-repository ppa:chris-lea/node.js
~~~~

Finally, install node.js:

~~~~bash
sudo apt-get install nodejs
~~~~

Our droplet is now provisioned with node.js!

The final step for getting our droplet up and running is declaring our application as a service that can be stopped and (re)started. We will be using [upstart](http://upstart.ubuntu.com/) for this as it comes with Ubuntu.

Create a file called `node-app.conf` in `/etc/init/` on your droplet with the following contents:

~~~~
#!upstart

description "node-app"
author      "mies"

start on (local-filesystems and net-device-up IFACE=eth0)
stop  on shutdown

respawn                # restart when job dies
respawn limit 5 60    # give up restart after 5 respawns in 60 seconds

nice 5

script
  node /var/local/www/app.js
end script
~~~~

The most import part here is the `script` section that starts our node.js application. Of course, we want to be able to restart our application when we make modifications to it and deploy new versions. We'll set this up through our deployment pipeline on wercker.

## Updating our wercker.yml for deployment

Now that we have created a deploy target on wercker, have set up communication through SSH and of course have provisioned our droplet we're ready to set up our deployment pipeline on wercker. We will leverage various [steps](http://devcenter.wercker.com/articles/steps/) available in the [wercker directory](https://app.wercker.com/#explore) to make our life easier.

Update your [wercker.yml file](http://devcenter.wercker.com/articles/werckeryml/) with the following contents.

~~~~yaml
box: wercker/nodejs
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
deploy:
  steps:
    - add-to-known_hosts:
        hostname: mies.io
    - mktemp:
        envvar: PRIVATEKEY_PATH
    - create-file:
        name: write key
        filename: $PRIVATEKEY_PATH
        content: $WERCKER_PRIVATE
        overwrite: true
    - script:
        name: transfer application
        code: |
          pwd
          ls -la
          scp -i $PRIVATEKEY_PATH app.js package.json root@mies.io:/var/local/www
    - script:
        name: npm install
        code: ssh -i $PRIVATEKEY_PATH -l root mies.io "cd /var/local/www/ &&  npm install --production"
    - script:
        name: start application
        code: |
          ssh -i $PRIVATEKEY_PATH -l root mies.io "if [[ \"\$(status node-app)\" = *start/running* ]]; then stop node-app -n ; fi"
          ssh -i $PRIVATEKEY_PATH -l root mies.io start node-app
~~~~

First, we need to add the server we want to deploy to, to the known hosts file. We can do this using [the add-to-know_hosts step](https://app.wercker.com/#applications/521764dde36a64ff110022f2/tab/details).

Second, we create a temporary file and make it available as an environment variable using the [mktemp step](https://app.wercker.com/#applications/52167277e9fa619606001064/tab/details). This filepointer will contain our private key that we've previously created.

Next, we write the contents of our private key ($WERCKER_PRIVATE) to the temporary file we created in the previous step using mktemp. We use the [create-file step](https://app.wercker.com/#applications/51c829dd3179be4478002113/tab/details) to do so. Now we are ready to communicate with our droplet!

I transfer my application in the following step, consisting of `package.json` and the `app.js` file using scp to my remote /var/local/www/ folder.

I need to install the dependencies in the `package.json` file using npm which I do in the subsequent step.

Finally, I check through upstart if my application is running. If this is the case I stop it, and then restart it.

Don't forget to commit and push the updated `wercker.yml` file:

~~~~bash
git commit -am 'updated wercker.yml with deploy pipeline'
git push origin master
~~~~

You are now able to pick your latest green build from wercker and deploy it.

![image](http://f.cl.ly/items/2e1O212I1C1o1N2W1c0F/Screen%20Shot%202013-09-03%20at%202.28.06%20PM.png)

This will trigger a deploy and wercker will go through the **deploy** section of your wercker.yml file, which looks as follows:

![image](http://f.cl.ly/items/0y2E0r212P3f1H1q0p2M/Screen%20Shot%202013-09-03%20at%202.11.52%20PM.png)

Congratulations you've succesfully set up your continuous delivery pipeline with wercker and Digital Ocean.

If you visit your droplet you should see our clouds!

![image](http://f.cl.ly/items/160G1g0L3a3q1g2w1R2o/Screen%20Shot%202013-09-03%20at%202.42.02%20PM.png)

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.
