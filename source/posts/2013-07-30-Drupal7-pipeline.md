---
title: Drupal7 pipeline by Desmond Morris
date: 2013-07-30
tags: php, drupal
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
Our friend <a href="http://www.desmondmorris.com/">Desmond Morris</a>
who heads up engineering at <a href="http://www.dosomething.org/">Do
Something</a> created a wercker <a href="">pipeline</a> specifically for
Drupal7.
</h4>

We recently added [php
support](http://blog.wercker.com/2013/07/01/Announcing-php-support.html)
to wercker which allows you to build and deploy PHP applications, for
instance those created with [Drupal](http://drupal.org), a powerful open
source content management system.

We've documented wercker's php support [at our
devcenter](http://devcenter.wercker.com/articles/languages/php.html)
with a getting started guide
[here](http://devcenter.wercker.com/articles/languages/php/gettingstarted-php.html).

![image](http://f.cl.ly/items/1W1q1K2n3X0L3u0K3x1h/wercker%2Bdrupal.png)

READMORE

Desmond created a
[wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) that
leverages [drush](https://drupal.org/project/drush) the drupal command
line tool.

You can find his
[wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file below:

```yaml
box: wercker/php
services:
    - wercker/mysql
build:
    steps:
    -  script:
          name: Install Drush
          code: |-
              pear channel-discover pear.drush.org
              pear install drush/drush
              phpenv rehash
    -  script:
          name: Install Drupal
          code: |-
              drush site-install standard --db-url=$WERCKER_MYSQL_URL --site-name=Testing -y
    -  script:
          name: Enable simpletest
          code: |-
              drush en simpletest -y
              drush vset --yes simpletest_verbose FALSE
    -  script:
          name: Start server
          code: |-
              drush runserver --server=builtin 8080 & sleep 5
    -  script:
          name: Run tests
          code: |-
              drush test-run DrupalGotoTest --xml --uri=http://127.0.0.1:8080
```

## Drupal 7 wercker.yml

The wercker [php](http://devcenter.wercker.com/articles/boxes/)
[box](https://app.wercker.com/#explore/boxes/wercker/php) comes with
[Pear](http://pear.php.net/) which is used to install
the drush commandline tool in the first step.

Next, Drupal7 is installed with the `WERCKER_MYSQL_URL` which is
available, as the **wercker/mysql** [service](http://devcenter.wercker.com/articles/services/) is defined at the top.

Simpletest and the development server are stated in the next two steps.

Finally the tests are run, again using Drush.

A simple yet powerful wercker pipeline for Drupal applications!

Thanks goes out to [Desmond](https://twitter.com/desmondmorris) for sharing his wercker.yml with everyone!

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.



