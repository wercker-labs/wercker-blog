---
title: Developer Contributions!
date: 2013-06-26
tags: wercker, developers, evangelism
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
We've had some awesome developer contributions by <a href="http://twiter.com/wunki">Petar Radošević</a> and <a href="https://twitter.com/jkeyes">John Keyes</a> this week and we'd love to share them with the rest of the community. Petar got <a href="http://clojure.org/">Clojure</a> working with wercker, while John created a <a href="http://wordpress.org">wordpress environment</a> on wercker.
</h4>

READMORE

### Wordpress and wercker

Although wercker has no support for the php programming language yet (more on this soon!), the [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) build pipeline is extremely flexible. [John Keyes](http://keyes.ie) took this flexibility and created a [wordpress](http://wordpress.org) box alongside Apache.

This is the `wercker.yml` John created to make this happen:

``` yaml
box: wercker/default
services:
    - wercker/mysql
build:
  steps:
    -  script:
        name: install WordPress
        code: |-
          # install PHP 5 with MySQL support, and Apache with PHP support.
          sudo apt-get update
          sudo apt-get install php5 php5-mysql
          sudo apt-get install libapache2-mod-php5
          sudo a2enmod php5
          sudo service apache2 restart
          # install WordPress
          tar zxf wordpress-3.5.2.tar.gz
          sudo mv wordpress/* /var/www/
          # replace tokens with values from Wercker ENV
          sed -e "s/{{DB_NAME}}/$WERCKER_MYSQL_DATABASE/" -e "s/{{DB_USER}}/$WERCKER_MYSQL_USERNAME/" -e "s/{{DB_PASSWORD}}/$WERCKER_MYSQL_PASSWORD/" -e "s/{{DB_HOST}}/$WERCKER_MYSQL_HOST:$WERCKER_MYSQL_PORT/" < wp-config.template > wp-config.php
          sudo mv wp-config.php /var/www/wp-config.php
          # delete the default index file
          sudo rm /var/www/index.html
          # POST the install variables
          curl -X POST --data "weblog_title=test&user_name=admin&admin_password=admin&admin_password2=admin&admin_email=admin@example.com" -i http://localhost/wp-admin/install.php?step=2
          # validate the install worked
          curl http://localhost | grep "Welcome to WordPress. This is your first post."
```

You can check out his project on [GitHub](https://github.com/jkeyes/wercker-wordpress) and on [wercker](https://app.wercker.com/#project/51c772c43179be44780012e9).

### Clojure and wercker

Like [Gibbon's](http://gibbon.co) Petar Radošević aka [@wunki](http://twiter.com/wunki), we are a big fan of functional programming languages such as Erlang and Scala. Petar created a [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) that provisions a box containing [Clojure](http://clojure.org/) alongside [Leiningen](https://github.com/technomancy/leiningen), complete with unit-testing and packaging of a `jar` with all dependencies, using `lein uberjar`.

``` yaml
box: wercker/default
services:
  - wercker/postgresql
  - wercker/rabbitmq
  - wercker/redis
build:
  steps:
    - script:
        name: install clojure
        code: |
          sudo apt-get update
          sudo apt-get install openjdk-7-jdk curl -y
          sudo wget -O /usr/local/bin/lein https://raw.github.com/technomancy/leiningen/stable/bin/lein
          sudo chmod +x /usr/local/bin/lein
    - script:
        name: run tests
        code: |
          lein test
    - script:
        name: build
        code: |
          lein uberjar
```

You can find a gist of this wercker.yml that you can fork [here](https://gist.github.com/wunki/5826444). Read more about the `wercker.yml` format on our [dev center](http://devcenter.wercker.com/articles/werckeryml/).

### Thanks!

Thanks to both Petar and John for these awesome contributions and we'll be sending some extra [wercker stickers](https://twitter.com/Wercker/status/349481467916734464/photo/1) your way ;-)

Also, make sure to follow this blog for some exciting announcements surrounding boxes.