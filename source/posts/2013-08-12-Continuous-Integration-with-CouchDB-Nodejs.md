---
title: Continuous Integration with CouchDB, Nodejs and wercker
date: 2013-08-12
tags: couchdb, nodejs
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
We've previously written about how we created a <a href="http://blog.wercker.com/2013/07/24/Building-your-own-box-with-Chef.html">CouchDB Service box</a> for your wercker <a href="http://devcenter.wercker.com/articles/introduction/pipeline.html">pipelines</a>. We got some questions on how to leverage this box so in this article we are going to showcase how to do some continuous integration with the CouchDB box and node.js.
</h4>

You can find the code for this sample application on [GitHub](https://github.com/mies/getting-started-couchdb). View my application and its status on wercker [here](https://app.wercker.com/#applications/51cb0a0ec3f3032f38000067).

[![wercker status](https://app.wercker.com/status/21e01d1e73b3f3e230a920e6eab2ef80/m "wercker status")](https://app.wercker.com/project/bykey/21e01d1e73b3f3e230a920e6eab2ef80)

## Declaring our dependencies

Let's start out by declaring our dependencies in node.js' **package.json** format:

``` javascript
{
  "name" : "getting-started-couchdb",
  "version" : "0.0.1",
  "dependencies" : {
    "nano": "4.1.1"
  },
  "scripts": {
    "test" : "node app.js",
    "start": "app.js"
  }
}
```

The most important section in this file (apart from the dependency which is a node.js CouchDB driver) is the **scripts** part where we declare that when `npm test` is invoked our application should be started. As we are not writing a unit test for our application but just some code to test against CouchDB, we just run our app.

## Using the CouchDB box

You can find the source of the CouchDB box on [GitHub](https://github.com/mies/box-couchdb) and its wercker page
[here](https://app.wercker.com/#applications/51cace444b940c9e19004ba2/tab/details).

The [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) is the way to declare your build and deployment environment on wercker. For using node.js and CouchDB on wercker your **wercker.yml** should roughly look as follows:

``` yaml
# Use the wercker nodejs box
box: wercker/ubuntu12.04-nodejs0.10@0.0.2
services:
  - mies/couchdb
# Build definition
build:
  steps:
    # install dependencies
    - npm-install
    # test our couchdb box by create a 'baseball' database and then retrieve all the database names
    - script:
        name: curl
        code: |
          echo $WERCKER_COUCHDB_HOST
          curl -X PUT "$WERCKER_COUCHDB_HOST:5984/baseball"
          curl -X GET "$WERCKER_COUCHDB_HOST:5984/_all_dbs"
    # Run our test application
    - npm-test
```

At the top you can see that we're using the [wercker nodejs 0.10 box](https://app.wercker.com/#applications/51cd71ca7578aa5b53000a2d/tab/details), this is the environment our build pipeline will run in. Next we specify that we want to use my [CouchDB box](https://app.wercker.com/#applications/51cace444b940c9e19004ba2/tab/details) as a service. This will spin up a seperate environment provisioned with CouchDB.

Next, we declare our build pipeline; we install the node.js dependencies (*in this case a node couchdb driver called nano*), run some **curl** calls to test our CouchDB service and connectivity and finally run **npm-test** that just runs our small app that we will discuss in a bit.

## Writing our test application

Now lets write a small piece of code to showcase the testing of our CouchDB box and how you could structure your own integration tests. We leverage the excellent [Nano library](https://github.com/dscape/nano), written by [Nuno Job](https://twitter.com/dscape). Create a file called **app.js** with the following contents:

``` javascript
var nano = require('nano')('http://' + process.env.WERCKER_COUCHDB_HOST + ':5984');
var db_name = "test";
var db = nano.use(db_name);

function insert_doc(doc, tried) {
  db.insert(doc,
    function (error,http_body,http_headers) {
      if(error) {
        if(error.message === 'no_db_file'  && tried < 1) {
          // create database and retry
          return nano.db.create(db_name, function () {
            insert_doc(doc, tried+1);
          });
        }
        else { return console.log(error); }
      }
      console.log(http_body);
  });
}

insert_doc({nano: true}, 0);
```
We connect to the CouchDB service box through the `WERCKER_COUCHDB_HOST` environment variable (which the box provides). Next we try to insert a hash, and if the database does not exists, create it. We also log any error or succes message. If you push this out to your version control system of choice (wercker supports GitHub and Bitbucket), wercker will run a build which should look as follows:

![image](http://f.cl.ly/items/0N173d3f2B1V3j2i3x34/Screen%20Shot%202013-08-12%20at%202.53.53%20PM.png)

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.