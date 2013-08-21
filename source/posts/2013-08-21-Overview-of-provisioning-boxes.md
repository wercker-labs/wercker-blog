---
title: Overview on how to provision wercker boxes
date: 2013-08-21
tags: provisioning, boxes, opendelivery
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
published: false
---

<h4 class="subheader">
The <a href="http://blog.wercker.com/2013/07/22/Announcing-the-Open-Delivery-platform.html">Open Delivery platform</a> which we've recently announced allows you to create your own environments on wercker, which run your build and deployment pipelines. We call these environments <a href="http://devcenter.wercker.com/articles/boxes/">boxes</a>. These <strong>boxes</strong> could run your language environment such as <a href="https://app.wercker.com/#applications/51ab917fdf8960ba45004497/tab/details">Ruby</a> or <a href="https://app.wercker.com/#applications/51f2cbaddf5a46247c0017c8/tab/details">Scala<a/>, or a service, like <a href="https://app.wercker.com/#applications/51cc29f604a2788145000b67/tab/details">RethinkDB</a> or <a href="https://app.wercker.com/#applications/51acf2a7c67e0560780006d2/tab/details">RabbitMQ</a>.
</h4>

In this post we'd like to go over two additional methods for creating these boxes.

![image](http://f.cl.ly/items/240Z2W073i0n1n1M3v1Z/wercker%2Bprovisioning.png)

READMORE

We've previously written tutorials on provisioning your boxes with [Chef](http://devcenter.wercker.com/articles/boxes/chef.html), [Puppet](http://devcenter.wercker.com/articles/boxes/puppet.html) and simple [Bash-based](http://devcenter.wercker.com/articles/boxes/bash.html) scripting.

Next to these provisioning systems we've also added both [Ansible](www.ansibleworks.com) and [Salt Stack](http://saltstack.com/).

## Ansible

In order to create a box provisioned with Ansible, you would inherit from the [wouter/ubuntu12.04-ansible](https://app.wercker.com/#applications/520b5e058a20a2624500a8f8/tab/details) box that is capable of running Ansible [playbooks](http://www.ansibleworks.com/docs/playbooks.html). In the your [wercker-box.yml](http://devcenter.wercker.com/articles/boxes/#create) you would do that as follows:

``` yaml
box: wouter/ubuntu12.04-ansible
```

You would then provision your your box by running a playbook. As we are executing our playbook on the same box that needs to be provisioned we need to define our host as localhost in our inventory but also pass the `-c local` flag when running our playbook. Your inventory file should look as follows:

**inventory.ini**

``` bash
[localhost]
127.0.0.1
```

As a trivial example we will be provisioning this box with **vim**, so our playbook would look roughly as follows:

**site.yml**

``` yaml
---
- hosts: localhost
  tasks:
    - name: ensure vim is at the latest version
      apt: pkg=vim state=latest
```

In your **wercker-box.yml** you would then run the playbook as follows in the `script` step that actually provisions the box:

**wercker-box.yml**

``` yaml
script: |
    sudo ansible-playbook -v site.yml -i inventory.ini -c local
```

We've created a sample app that installs **vim** via Ansible provisioning [here](https://github.com/mies/getting-started-ansible).
Note this is not a complete box.


## Salt Stack
Similar to Ansible and Chef-solo you would provision your wercker boxes in a masterless fashion with Salt Stack, as explained in their documentation [here](http://docs.saltstack.com/topics/tutorials/quickstart.html).

Again, you would inherit from a box that is capable of running salt modules which is the [wouter/ubuntu12.04-saltstack](https://app.wercker.com/#applications/5213619ccd37be2d020006dc/tab/details) box.

In your [wercker-box.yml](http://devcenter.wercker.com/articles/boxes/#create), the script step that installs the Salt Stack modules would look as follows:

**wercker-box.yml**

``` yaml
script: |
    sudo salt-call --local pkg.install nano refresh=true
    sudo salt-call --local pkg.file_list nano
```

It's as simple as that!

We've created a sample app that installs **nano** from a Salt Stack module [here](https://github.com/wwwouter/getting-started-saltstack/).
Note this is not a complete box.

## Other ways of creating boxes:

Check out the [wercker dev center](http://devcenter.wercker.com) for articles on creating your [wercker boxes](http://devcenter.wercker.com/articles/boxes/) with:

* [Bash](http://devcenter.wercker.com/articles/boxes/bash.html)
* [Chef](http://devcenter.wercker.com/articles/boxes/chef.html)
* [Puppet](http://devcenter.wercker.com/articles/boxes/puppet.html)

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.