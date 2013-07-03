---
title: Announcing PHP support for wercker
date: 2013-07-01
tags: php, announcement
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
Today, we're very excited to announce <a href="http://php.net/">PHP</a> support for wercker! PHP is of course a popular scripting language that powers for instance the <a href="http://drupal.org">Drupal CMS</a> and is the programming language that backs <a href="http://wikipedia.org">Wikipedia</a> and <a href="http://facebook.com">Facebook</a>.
</h4>

![image](http://f.cl.ly/items/190T2x2r463e020g182U/wercker%2Bphp.png)

READMORE

### Getting started with PHP and wercker

As always; signing up for wercker is [free and easy](https://app.wercker.com/users/new/).

There are three versions available on the wercker PHP box. The previous stable release PHP 5.3, the current stable release PHP 5.4 and the upcoming release PHP 5.5. By default the current stable release 5.4 is active. In terms of package managers, the following tools are installed [PEAR](http://pear.php.net/), [Pyrus](http://pear.php.net/manual/en/pyrus.about.php) and [Composer](http://getcomposer.org/). You can read more about leveraging these package managers on our [dev center](http://devcenter.wercker.com/articles/languages/php.html).

##### Update: we have added a tutorial to get started with wercker and PHP on our [dev center](http://devcenter.wercker.com/articles/languages/php/gettingstarted-php.html)

### wercker.yml

The [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) is the way to define your build and deployment pipeline on wercker. Doing so for PHP is trivial:

``` yaml
box: wercker/php
build:
  steps:
    - script:
        name: List available PHP versions
        code: |-
            phpenv version
            phpenv versions
            php -v
            which php
```

A more complete example that also installs php extensions using [pecl](http://pecl.php.net/), runs your application server and executes [phpunit](http://phpunit.de/manual/current/en/index.html) is again available on our [dev center](http://devcenter.wercker.com/articles/languages/php.html).

We have also created a sample application that you can fork and test with wercker [here](https://github.com/wercker/getting-started-php).

### Earn some stickers!

Tell us about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).