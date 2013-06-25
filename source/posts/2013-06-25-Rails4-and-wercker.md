---
title: Rails 4 and wercker
date: 2013-06-25
tags: wercker, ruby, rails
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
Rails4 was just <a href="http://weblog.rubyonrails.org/2013/6/25/Rails-4-0-final/">released</a> and we briefly want to showcase
how to test and deploy your Rails4 application with <a href="http://wercker.com">wercker</a>.
</h4>

READMORE

### Getting started

Signing up for wercker is easy and free, [so go ahead](https://app.wercker.com/users/new/). You can check out the code for this tutorial on [GitHub](https://github.com/mies/getting-started-rails4), feel free to fork and clone this app.

Explore this application on wercker [here](https://app.wercker.com/#project/51c9b6524b940c9e190033d3).

We will use the wercker command line interface

First, make sure you have Rails4 installed on your machine. Check out the announcement for more info, but the gist of it is:

``` bash
gem install rails --version 4.0.0 --no-ri --no-rdoc
```

Next let's create a new application:

``` bash
rails new getting-started-rails4
```

### Declaring our dependencies

Now let's declare our dependencies in the Gemfile. Rails 5 will require Ruby version `2.0.0` so let's get our feet wet with the newest Ruby as well.

Modify your `Gemfile` in the following way:

``` ruby
source 'https://rubygems.org'

ruby '2.0.0'

gem 'unicorn'
```

Also substitute the `sqlite` gem with `pg` as we will be using Postgres on [wercker](http://wercker.com).

``` ruby
gem 'pg'
```

### Adding our Heroku deploy target

We want to deploy to [Heroku](http://heroku.com), so first create a `Procfile`:

``` bash
web: bundle exec unicorn -p $PORT -c ./config/unicorn.rb
```
We're using the [unicorn](http://unicorn.bogomips.org/) application server so let's create a configuration file for it in the `config` folder:

``` ruby
# config/unicorn.rb
worker_processes Integer(ENV["WEB_CONCURRENCY"] || 3)
timeout 15
preload_app true

before_fork do |server, worker|
  Signal.trap 'TERM' do
    puts 'Unicorn master intercepting TERM and sending myself QUIT instead'
    Process.kill 'QUIT', Process.pid
  end

  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.connection.disconnect!
end

after_fork do |server, worker|
  Signal.trap 'TERM' do
    puts 'Unicorn worker intercepting TERM and doing nothing. Wait for master to send QUIT'
  end

  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.establish_connection
end
```

Next we create a Heroku application:

``` bash
heroku create

Creating powerful-badlands-2653... done, stack is cedar
http://powerful-badlands-2653.herokuapp.com/ | git@heroku.com:powerful-badlands-2653.git
Git remote heroku added
```
****
##### Note wercker also has an add-on available for Heroku on the [marketplace](https://addons.heroku.com/wercker), for which you have to be part of the [Heroku Beta program](http://beta.heroku.com)
****

### Setting up our wercker.yml file

The [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file is a simple but powerful way to set up your build and deployment pipeline. Add a file called `wercker.yml` to your repository with the following contents:

``` yaml
box: wercker/ubuntu12.04-ruby2.0.0
services:
  - wercker/postgresql
build:
  steps:
    - bundle-install
    - rails-database-yml
```
We first specify the Ruby box that we want to use on wercker, which is of course the `2.0.0` box. Next we specify the services that we want to leverage. As said, we will be using Postgresql for our database backend, so we add it to our wercker.yml. We subsequently run the `bundle-install` step that will run a `bundle install` command to install your gems. Next, the `rails-database-yml` step generates a `database.yml` file that will set up the wercker dependent connection strings for your database.

Similar to [Heroku](https://devcenter.heroku.com/articles/heroku-postgresql#connecting-in-rails), there is [no need](https://devcenter.heroku.com/articles/ruby-support#build-behavior) to specify your database connections in a `database.yml` file as one is generated for you, based on the services section in your `wercker.yml` file and the `rails-database-yml` build step, each time a build is run on wercker.

Commit and push your changes to [GitHub](http://github.com) or [Bitbucket](http://bitbucket.org).

### Adding our application to wercker

You can now either add your application to wercker via the [command line interface](http://devcenter.wercker.com/articles/gettingstarted/cli.html) or via the [web ui](http://devcenter.wercker.com/articles/gettingstarted/web.html). We will be using the [CLI](http://devcenter.wercker.com/articles/cli/) which is a Python application that you can install via:

``` bash
pip install wercker
```

No we add our app to wercker by running:

``` bash
wercker create
```

That will not only add our application to wercker but also automatically set up our Heroku deploy target and trigger a build!
Let's view our application on wercker by running (or just visit the web interface)

``` bash
wercker open
```
You should now see your green build!

![image](http://f.cl.ly/items/2F2f0G0o0a0o403E2p1o/Screen%20Shot%202013-06-25%20at%206.08.04%20PM.png)

Finally, running the following command will deploy your application to Heroku:

``` bash
wercker deploy
```

This command will ask you which build you want to deploy to which target (we just have one on Heroku) and presents the following output:

```bash
-----------------------
welcome to wercker-cli
-----------------------

Retreiving builds from wercker...
Found 4 result(s)...

┌───┬────────┬──────────┬────────┬──────────┬───────────────────┬────────────────┐
│   │ result │ progress │ branch │ hash     │ created           │ message        │
├───┼────────┼──────────┼────────┼──────────┼───────────────────┼────────────────┤
│ 1 │ passed │ 100.0%   │ master │ 31f76574 │ 06/25/13 15:51:15 │ added unicorn  │
├───┼────────┼──────────┼────────┼──────────┼───────────────────┼────────────────┤
│ 2 │ passed │ 100.0%   │ master │ dacad4be │ 06/25/13 15:47:41 │ added readme   │
├───┼────────┼──────────┼────────┼──────────┼───────────────────┼────────────────┤
│ 3 │ passed │ 100.0%   │ master │ fd070ed5 │ 06/25/13 15:32:21 │ added Procfile │
├───┼────────┼──────────┼────────┼──────────┼───────────────────┼────────────────┤
│ 4 │ passed │ 100.0%   │ master │ 3cf7020e │ 06/25/13 15:25:08 │ init           │
├───┼────────┼──────────┼────────┼──────────┼───────────────────┼────────────────┤
Select which build to deploy(enter=1):

Retreiving list of deploy targets...
Found 1 result(s)...

┌───┬─────────┬────────┬───────────┬───────────────────┬────────┬──────────┬────────────────┐
│   │ target  │ result │ deploy by │ deployed on       │ branch │ commit   │ message        │
├───┼─────────┼────────┼───────────┼───────────────────┼────────┼──────────┼────────────────┤
│ 1 │ staging │ passed │ mies      │ 06/25/13 15:47:27 │ master │ fd070ed5 │ added Procfile │
├───┼─────────┼────────┼───────────┼───────────────────┼────────┼──────────┼────────────────┤
Select a target to deploy to(enter=1):
Success:
            Build scheduled for deploy.

You can monitor the scheduled deploy in your browser using:
wercker targets deploy
Or query the queue for this application using:
wercker queue
```
Your application should now be alive and kicking on Heroku!

We briefly discussed how to test and deploy your Rails4 application with wercker. Check out our [dev center](http://devcenter.wercker.com) for more information and other guides

### Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy]((https://app.wercker.com/users/new/).