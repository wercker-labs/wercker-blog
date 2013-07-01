---
title: Wercker.yml for Rails 4, Postgres and Heroku
date: 2013-07-01
tags: rails4, ruby, heroku
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
Right on the heels of our <a href="http://blog.wercker.com/2013/06/25/Rails4-and-wercker.html">post</a> on how to use <a href="http://weblog.rubyonrails.org/2013/6/25/Rails-4-0-final/">Rails4</a> and <a href="http://wercker.com">wercker</a>, <a href="https://twitter.com/frunns">Frans Krojegård</a> was kind enough to share his <a href="http://devcenter.wercker.com/articles/werckeryml/">wercker.yml</a> file that showcases a complete build and deployment pipeline on wercker!
</h4>

As always, signing up for wercker is [free and easy](https://app.wercker.com/users/new/).

READMORE

Frans' wercker.yml file for his [Rails4](http://www.infoq.com/news/2013/06/rails4) application, uses the wercker Ruby 2.0 [box](https://github.com/wercker/box-ubuntu12.04-ruby2.0.0), a Postgres [service](http://devcenter.wercker.com/articles/services/) and contains [Rake](http://rake.rubyforge.org/) task that bootstraps his database. Finally, he runs his rspec tests using `bundle exec rspec`. For deployment, he leverages the `heroku-deploy` step, after which he runs a database migration script. You can view his complete [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file below or check out the [gist](https://gist.github.com/frunns/5895115).

``` yaml
box: wercker/ubuntu12.04-ruby2.0.0
services:
    - wercker/postgresql
build:
    steps:
        - bundle-install

        - rails-database-yml:
            service: postgresql

        - script:
            name: echo ruby information
            code: |
                echo "ruby version $(ruby --version) running!"
                echo "from location $(which ruby)"
                echo -p "gem list: $(gem list)"

        - script:
            name: Set up db
            code: bundle exec rake db:schema:load RAILS_ENV=test

        - script:
            name: rspec
            code: bundle exec rspec

deploy:
    steps:
        - heroku-deploy
        - script:
            name: Update database
            code: heroku run rake db:migrate --app $TARGET_NAME
```

Thanks goes out to [Frans](https://twitter.com/frunns) for sharing his wercker.yml, we'll be sending some extra wercker stickers his way soon ;-)

![image](http://f.cl.ly/items/3v3U2f0u1H062Q2D3t0g/BNmcElCCcAE7Vy9.jpg)

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Thanks for tuning in, and there will be more posts next week! Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.
