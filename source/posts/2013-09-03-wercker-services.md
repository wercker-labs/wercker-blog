---
title: Improved support for services
date: 2013-09-03
tags: wercker, cli, services, boxes
author: Jacco Flenter
gravatarhash: 7d9ef3d3f6911e6e4f9c51f6d99c48f8
published: false
---

<h4 class="subheader">
A while ago we introduced the concepts of boxes on the wercker platform, which allows you to create your own environments and services (think CouchDB/MongoDB or even RabbitMQ). 
We have just made it easier to add any service to your wercker app.
</h4>

READMORE

This week we've added two improvements to the wercker toolchain. First of all, you can now easily see in our [overview of boxes](https://app.wercker.com/#/explore/boxes) what boxes or services are available (note: you can also use the filter to show only services).

![image](http://cl.ly/RBkE/Screen%20Shot%202013-09-03%20at%205.24.32%20PM.png)

The second improvement is to the [wercker cli](http://devcenter.wercker.com/articles/cli/installation.html), it now supports a range of service related commands:

``` text
    services                            - list services specified in the
                                          wercker.yml
    services add <name> [<version>]     - add a service to your
                                          wercker.yml
    services remove <name>              - remove a service from your
                                          wercker.yml
    services search [<name>]            - find a service
    services info <name> [<version>]    - display detailed information for
                                          service
```

The services commands can help you find services, modify your [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) and show if you are using the latest version or not. One small remark: if you have no wercker.yml yet, go to a build and view the **setup environment step**. In the log you can see the wercker.yml that wercker generated for you.

### Searching ###
So imagine you want to add CouchDB to your existing application, simply run `wercker services search couch` from somewhere within your project folder:

``` bash
$ wercker services search couch
-----------------------
welcome to wercker-cli
-----------------------

mies/couchdb                   - 0.1.3-dev - wercker box for couchdb a document oriented database
```

We get one result for CouchDB. So let's get some more details through the use of `wercker services info mies/couchdb`.

``` bash

    $ wercker services info mies/couchdb
    -----------------------
    welcome to wercker-cli
    -----------------------

    Retrieving service: mies/couchdb

    Fullname:           mies/couchdb
    Owner:              mies
    Name:               couchdb

    All versions:
    0.0.1
    0.0.2
    0.0.9
    0.1.0
    0.1.1
    0.1.2
    0.1.3-dev
    Latest version:     0.1.3-dev

    License:            none specified
    Keywords:           couchdb, erlang, noqsql, database

    Description:
    wercker box for couchdb a document oriented database

    Packages:
    None specified

    Read me:
    box-couchdb
    ===========

    Wercker box for couchdb, a document oriented database.
```

### Adding and updating ###
The result earlier is truncated since it is not relevant to show all the details on CouchDB's service on wercker for this article. However you may decide to use version 0.1.2, since the latest version is a prerelease with its -dev marking. Adding it like so: 

``` bash
wercker services add mies/couchdb 0.1.2
```

Now let's validate the result by running `wercker services`

``` bash
    $ wercker services
    -----------------------
    welcome to wercker-cli
    -----------------------

    Services currently in use:

    mies/couchdb - 0.1.2 (upgrade to 0.1.3-dev) - wercker box for couchdb a document oriented database
```


And we can see there's a newer version of couchdb, which we can upgrade by simply running `wercker services add mies/couchdb` or maybe just specify we want a version newer than 0.1.2: `wercker services add mies/couchdb ">0.1.2"`.

We hope you enjoy this update to the wercker cli, which you can install by running:

``` bash
pip install wercker
```
Have fun!

### Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).
