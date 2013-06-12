---
title: Getting started with Flask, Redis and wercker
date: 2013-06-11
tags: flask, redis, wercker, python
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
In this post we'll dive into how to get started with a simple <a href="http://flask.pocoo.org">Flask</a> application with a <a href="http://redis.io">redis</a> backend and how to test it with wercker.
</h4>

Flask is a powerful microframework for Python web applications. It has a rich ecosystem of plugins and snippets and is of course open source! Flask is ideal for building lightweight applications or API's. It is also *very* easy to learn.

READMORE

[Redis](http://redis.io) is an in-memory store for datastructures such as lists, sets and hashes. It is also very fast!

In this tutorial we are going to build a very simple API that returns a list of latin *cloud* names in JSON which are retrieved from Redis. We're also going to add a unittest to test the functionality of our API. Finally we will test all of this within wercker, so lets get started! The code for this example application can be found on [GitHub](https://github.com/mies/getting-started-flask-redis). This guide is also available on our [dev center](http://devcenter.wercker.com/articles/languages/python/flaskredis.html)

You can see the final result of this application on my [wercker page](https://app.wercker.com/#project/51b6e0c4345a2a453d000bfa).

### Prerequisites


* A [GitHub](https://github.com/) or [Bitbucket](http://bitbucket.org) account with a pre-made repository that will hold your code
* You have [registered for a wercker account](https://app.wercker.com/users/new) and [signed in](https://app.wercker.com/users)
* Basic Python and Redis knowledge

### Setting up our virtualenv and requirements.txt

We *highly* recommend using [virtualenv](http://www.virtualenv.org/) and [pip](http://www.pip-installer.org/) to islolate your python environment, so lets do that. In your project folder run the following commands:

``` bash
$ virtualenv venv --distribute
$ source venv/bin/activate
```

This creates a virtual environment folder called 'venv' and activates the Python executable therein.

Next lets install our dependencies. In this case we need Flask and the [Python Redis interface](https://github.com/andymccurdy/redis-py):

``` bash
$ pip install flask
$ pip install redis
```
We want to save our requirements.txt file for our environment for reuse. Wercker also needs this file of course to run our test that we'll create later.

``` bash
$ pip freeze > requirements.txt
```

Our environment is set up, so lets get started building our API.

### Creating our Flask API.

Create a file called `app.py` in your project folder with the following contents:

``` python
import os
import redis

from flask import Flask
from flask import Response
from flask import json

app = Flask(__name__)
app.redis = redis.StrictRedis(host=os.getenv('WERCKER_REDIS_HOST', 'localhost'),
      port= 6379, db=0)

@app.route("/clouds.json")
def clouds():
  data = app.redis.lrange("clouds", 0, -1)
  resp = Response(json.dumps(data), status=200, mimetype='application/json')
  return resp

if __name__ == "__main__":
  port = int(os.getenv('PORT', 5000))
  app.run(host='0.0.0.0', port=port)
```

In this file you see that we import the 'redis' library, we also import the 'os' package as we need it to access environment variables and some Flask dependent imports such as the ability to dump JSON.

We set up a connection to Redis and attach it to our 'app' Flask instance. We use [environment variables](http://www.12factor.net/config) to either connect to the Redis instance as provided by wercker, or if we are running on 'localhost', we just simply use a Redis instance started (FYI, the 'redis-server' command) on our local machine.

Next, we create a route called `/clouds.json` that returns JSON. The data to return is retrieved from a Redis [list](http://redis.io/commands#list) that we subsequently dump to JSON.

Finally, we add the code to run our application and again we use the `PORT` environment variable that Platform-as-a-Service providers such as [Heroku](http://heroku.com) use, it is not necessary here, but good practice.

This app should run if we execute `python app.py`, but it will not return any data if we for instance `$ curl localhost:5000/clouds.json` because there is nothing stored in Redis yet. Lets do that through our unittest.

### Writing our unittest

We are now ready to test our cloud api. To do so, we create a seperate file that wil contain our unittest that we will write using the standard Python [unittest framework](http://docs.python.org/2/library/unittest.html). For more information on testing Flask applications, see [this section](http://flask.pocoo.org/docs/testing/) of the Flask documentation site.

Create the following `app_test.py`:

``` python
from app import app

import redis
import os
import unittest
import json

class CloudTestCase(unittest.TestCase):

  def setUp(self):
    app.redis.rpush('clouds','Altocumulus')
    app.redis.rpush('clouds','Altostratus')
    app.redis.rpush('clouds','Cumulonimbus')
    app.redis.rpush('clouds','Nimbostratus')

  def tearDown(self):
    app.redis.flushdb()

  def test_clouds(self):
    tester = app.test_client(self)

    response = tester.get('/clouds.json', content_type='application/json')

    self.assertEqual(response.status_code, 200)
    self.assertEqual(response.data, json.dumps(['Altocumulus', 'Altostratus',
      'Cumulonimbus', 'Nimbostratus']))

if __name__ == '__main__':
  unittest.main()
```

Again we import several libraries, of note are the `unittest` library, and the fact that we import our application from the file, `app.py`, we previously created.

We see *three* functions; one to setup our unittest, one to tear it down again and the actual test. In the `setUp` function we fill Redis with a list of several latin cloud names, in the `tearDown` function we flush the database to clean up after ourselves. Finally, the `test_clouds` function actually tests the HTTP response of our `/clouds.json` route in our application, checks if the HTTP status code return *200OK* and if the list of cloud names is the one we expected.

### Creating our wercker.yml file

Before we add our application to wercker there is one more thing we should do, which is setting up our build pipeline in such a way that wercker knows how to execute it. We do this through the creation of a *wercker.yml* file. For more information on wercker.yml see the [section](http://devcenter.wercker.com/articles/werckeryml/) on our developer center.

In your project folder create a file called `wercker.yml` with the following contents:

``` yaml
box: wercker/python
services:
  - wercker/redis
build:
  steps:
    - pip-install
    - script:
          name: flask tests
          code: python app_test.py
```
Lets briefly go through this file. First, you'll see the `box` definition that says `wercker/python`. This tells wercker that we want to run our code in a Python box (which makes total sense!). By default this [box](https://github.com/wercker/box-python) runs Python version 2.7.3. Next, we use the `services` clause to define that we want to make use of Redis on wercker by specifying `wercker/redis`. You can read more about other services such as MySQL, Postgres and RabbitMQ on the services section at our [dev center](http://devcenter.wercker.com/articles/services/). In return for specifying the `wercker/redis` service, we get several environment variables. We only need one of these environment variables; `WERCKER_REDIS_HOST` that we used in our `app.py` above for setting up our Redis connection.

Finally, you can see the `build` section, which contains two steps. First, the `pip-install` buildsteps that wraps the `pip install -r requirements.txt` command. Second, a custom buildstep defined via the `script` clause (a simple bash command). The step is named `flask tests` and runs the unit test that we previously created. You can leverage the custom step for any script you want to execute, for instance to *minify* your javascript or to compile your *sass* assets.

Now that we have created all the files we need, lets push this our to our version control system:

``` bash
$ git add .
$ git commit -am 'initial commit'
$ git push origin master
```
Now lets get started with wercker!

### Adding our application to wercker

We are now ready to add our repository to wercker so we can deploy safely when we create and push out new functionality for our application. Again make sure you have a wercker account, [sign up is free and easy](https://app.wercker.com/users/new/)!

After you have signed up, make sure you connect either your GitHub or Bitbucket account. Next you could add your repository throught the web interface by clicking the `add application button`. However, wercker also comes with a [command line interface](http://devcenter.wercker.com/articles/cli/) which is written in Python. As we're already developing a Python application we are going to use the `CLI` to add it to wercker.

The command line interface can be installed by running:

``` bash
$ pip install wercker
```

If you want to frequently use the CLI it is a good idea to install it outside of your 'virtual environment', otherwise it would only be available there.


Next we run:

``` bash
$ wercker login
```

This is only needed once and will ask you for your wercker username and password. Next we are ready to add our repository by running:

``` bash
$ wercker create
```

This command will do some introspection and see if it can find a git repository. It will also check to see if the user, `werckerbot` has access to the repository on GitHub or Bitbucket in order to run your builds. Make sure this is the case, and then go to your application page on wercker. A build has been triggered and you should see it running.

![image](http://f.cl.ly/items/1I3X3D3P2w0I290j3Q3J/Screen%20Shot%202013-06-11%20at%203.47.07%20PM.png)

When we click on the *flask tests* buildstep, we see that our unit test has passed.

![image](http://f.cl.ly/items/2X0S3z0Q1w1K2o2p2Q1Q/Screen%20Shot%202013-06-11%20at%204.46.08%20PM.png)

Congratulations, you've successfully created and tested a Flask application with wercker!

The code for this example application can be found on [GitHub](https://github.com/mies/getting-started-flask-redis).

You can see the final result of this application on my [wercker page](https://app.wercker.com/#project/51b6e0c4345a2a453d000bfa).