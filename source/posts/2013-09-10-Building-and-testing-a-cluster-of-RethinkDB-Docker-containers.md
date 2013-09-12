---
title: Building and testing a cluster of RethinkDB Docker containers
date: 2013-09-10
tags: docker, linuxcontainers, rethinkdb
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<!-- Auto-detect URL of current page and title if necessary -->
<a href="http://news.ycombinator.com/submit" class="hn-share-button">Vote on HN</a>

<!-- Override the URL and Title for the button -->
<a href="http://news.ycombinator.com/submit" class="hn-share-button" data-title="Building and testing a cluster of RethinkDB Docker containers" data-url="http://blog.wercker.com/2013/09/10/Building-and-testing-a-cluster-of-RethinkDB-Docker-containers.html">Vote on HN</a>

<h4 class="subheader">

    In the coming weeks we will be releasing more posts and screenshots on our experimental support for
<a href="http://docker.io">Docker</a> an exciting technology which is
an abstraction layer on top of <a
href="http://lxc.sourceforge.net/">LXC</a> allowing for lightweight "OS-level"
virtualized machines that are extremely portable.
Apply for early access that will allow you to build, test and deploy Docker containers from wercker.
</h4>

<div class="text-center">
<a href="http://wercker.com/docker/index.html#form" class="button radius secondary">Apply for our beta Docker support</a>
</div>

READMORE

## The Docker Registry

[dotCloud](https://www.dotcloud.com/), the company behind [Docker](http://docker.io) announced the
availabity of the [Docker Index](http://index.docker.io), a place where
you can find Docker images, for instance for your favorite [database](https://index.docker.io/u/mies/rethinkdb/) or
even [operating system](https://index.docker.io/_/ubuntu/).

Before pushing your Docker images to the Index you want to make certain that they work as expected.
Our experimental Docker support is ideally suited for not only building your Docker containers, but also for testing them.

In this tutorial we will build and test a container running [RethinkDB](http://rethinkdb.com), an open source database.
Let's say that you want to create a RethinkDB [cluster](http://rethinkdb.com/docs/start-a-cluster/) with a master and several slaves.
We'd want to make sure that we can correctly set up a cluster from our Docker image before deployment.

My Dockerfile for creating a RethinkDB image looks as follows:

``` bash
FROM base

maintainer Micha Hernandez van Leuffen

# install RethinkDB

run apt-get install -y software-properties-common
run add-apt-repository -y ppa:rethinkdb/ppa
run apt-get update
run apt-get install -y rethinkdb
```

This simply installs the packages from the RethinkDB PPA.

## Creating our build pipeline

Now that we have an image, we want to build and test it, which we of course do with the [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file.

For our experimental Docker support we leverage a customized version of wercker's [boxing system](http://devcenter.wercker.com/articles/boxes/). You can see my **wercker.yml** file below:

``` yaml
box: ami-7f97d816
box-type: ami
build:
  steps:
    - script:
        name: pip install
        code: |
          sudo apt-get update
          sudo apt-get install -y python-pip
          sudo pip install -r requirements.txt
    - script:
        name: build container
        code: sudo docker build -t mies/rethinkdb .
    - script:
        name: bootstrap containers
        code: source run.sh
    - script:
        name: run python test
        code: python test.py
```

After specifying the AMI that has Docker preinstalled I define my build pipeline.

* install pip itself and the dependencies which includes the RethinkDB python [driver](http://rethinkdb.com/docs/install-drivers/python/)
* build the container and name it `mies/rethinkdb`
* run a shell script to bootstrap the containers (we'll discuss this in a bit)
* run a python script that adds some data to our RethinkDB database and fetches it

## Bootstrapping the containers

I run a small shell script to bootstrap the RethinkDB cluster. The cluster consists of a master and two slaves.
In order to facilitate not only communication between the containers but also from my python script, we need to expose IP addresses and Port numbers,
which we do in the `run.sh` script that looks as follows:

``` bash
echo "---"
echo "setting up master"
echo "---"
export MASTER_ID=$(sudo docker run -p 29015:29015 -p 28015:28015 -d mies/rethinkdb rethinkdb --bind all)
export MASTER_IP=$(sudo docker inspect $MASTER_ID | grep IPAddress | cut -d '"' -f 4)
export MASTER_CLUSTER_PORT=$(sudo docker port $MASTER_ID 29015)
export MASTER_PORT=$(sudo docker port $MASTER_ID 28015)

echo "---"
echo "setting up first slave"
echo "---"
export SLAVE1_ID=$(sudo docker run -p 29015 -p 28015 -d mies/rethinkdb rethinkdb --join $MASTER_IP:$MASTER_CLUSTER_PORT --bind all)
export SLAVE1_PORT=$(sudo docker port $SLAVE1_ID 28015)
export SLAVE1_IP=$(sudo docker inspect $SLAVE1_ID | grep IPAddress | cut -d '"' -f 4)

echo "---"
echo "setting up second slave"
echo "---"
export SLAVE2_ID=$(sudo docker run -p 49235:29015 -p 49236:28015 -d mies/rethinkdb rethinkdb --join $MASTER_IP:$MASTER_CLUSTER_PORT --bind all)
export SLAVE2_PORT=$(sudo docker port $SLAVE2_ID 28015)
export SLAVE2_IP=$(sudo docker inspect $SLAVE2_ID | grep IPAddress | cut -d '"' -f 4)
```

We first create a Docker container running our master instance and export the container ID, ip, regular port and the port that is specific to cluster communication, as environment variables.
The `docker inspect` command is used to fetch the port and IP addresses.

Next, we create the two slaves and join them with the master using the **cluster** port.

The RethinkDB instances communicate over the IP addresses of the containers. You could also let the containers communicate over the host or via the container bridge and exposed ports.

## Testing our cluster

Now lets write a small python script that tests our RethinkDB cluster consisting of Docker containers.

``` python
import os
import rethinkdb as r

print 'Set up connection'

conn = r.connect(os.getenv('MASTER_IP'), os.getenv('MASTER_PORT')).repl()
print r.db_list().run(conn)
print conn.host

print 'Creating database'

r.db_create('wercker_tutorial').run(conn)

print 'Creating table'

r.db('wercker_tutorial').table_create('decepticons').run(conn)

print 'Inserting data'
r.db('wercker_tutorial').table('decepticons').insert({
      "decepticon": "Devastator",
      "team": "Constructicons",
      "members": ["Scrapper","Hook","Bonecrusher","Scavenger","Long Haul","Mixmaster"]
    }).run(conn)

print "Fetching data from the second slave..."
conn2 = r.connect(os.getenv('SLAVE2_IP'), 28015).repl()
print conn2.host

result = r.db('wercker_tutorial').table('decepticons').run(conn2)
for i in result:
  print i
```

The python script does the following:

* set up a connection to the master container, again using the exported environment variables
* create a database called 'wercker_tutorial'
* create a table called 'decepticons'
* insert some data in the master RethinkDB container
* fetch and display data from the second slave

The small test that we wrote passed as you can seen from the screenshot.

![image](http://f.cl.ly/items/2b2M1R2n2y0M09260c3g/Screen%20Shot%202013-09-10%20at%202.40.50%20PM.png)

We have tested the cluster capabilities of our RethinkDB Docker image and can now deploy, [share](http://blog.wercker.com/2013/09/06/Building-and-Storing-Docker-Containers.html) or push this image to the Docker index.

<!-- Auto-detect URL of current page and title if necessary -->
<a href="http://news.ycombinator.com/submit" class="hn-share-button">Vote on HN</a>

<!-- Override the URL and Title for the button -->
<a href="http://news.ycombinator.com/submit" class="hn-share-button" data-title="Building and testing a cluster of RethinkDB Docker containers" data-url="http://blog.wercker.com/2013/09/10/Building-and-testing-a-cluster-of-RethinkDB-Docker-containers.html">Vote on HN</a>


## Apply for our Docker beta

We've been hard at work at making delivery of Docker containers from wercker to various cloud providers, a reality.

Apply below for our Docker beta support and join us in making this happen.

Any additional awesome suggestions or feedback will be rewarded with some wercker stickers!

<div id="wufoo-z7x4m1">
Fill out my <a href="http://wercker.wufoo.com/forms/z7x4m1">online form</a>.
</div>
<script type="text/javascript">var z7x4m1;(function(d, t) {
var s = d.createElement(t), options = {
'userName':'wercker',
'formHash':'z7x4m1',
'autoResize':true,
'height':'679',
'async':true,
'header':'show'};
s.src = ('https:' == d.location.protocol ? 'https://' : 'http://') + 'wufoo.com/scripts/embed/form.js';
s.onload = s.onreadystatechange = function() {
var rs = this.readyState; if (rs) if (rs != 'complete') if (rs != 'loaded') return;
try { z7x4m1 = new WufooForm();z7x4m1.initialize(options);z7x4m1.display(); } catch (e) {}};
var scr = d.getElementsByTagName(t)[0], par = scr.parentNode; par.insertBefore(s, scr);
})(document, 'script');</script>

