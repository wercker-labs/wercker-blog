---
title: Packing and deploying Ruby apps with wercker and Capistrano
date: 2013-06-07
tags: ruby, bundler, capistrano, deployment
author: Benno van den Berg
gravatarhash: dff7a3e4eadab56aa69a24569cb61e98
---
<h4 class="subheader">
In this tutorial we’re going to show you the recommended way to package and deploy Ruby applications with wercker, bundler and capistrano. This article assumes you have some knowledge of how <a href="http://beta.wercker.com">wercker</a> works. This article is also available on our [dev center](http://devcenter.wercker.com/articles/deployment/capistrano.html).
</h4>

### Prerequisites

* Basic knowledge on Ruby and Capistrano
* You have [registered for a wercker account](https://app.wercker.com/users/new) and [signed in](https://app.wercker.com/sessions/new)
* You have [added your application to wercker](http://devcenter.wercker.com/articles/gettingstarted/web.html)

## The wercker.yml file

The *wercker.yml* file is the mechanism that allows you to define your environment on wercker; encompassing the programming language, any services you need such as databases and queues and of cource the build and deploy steps that specify the delivery pipeline for your application. For more information see the article on *wercker.yml* on our [dev center](http://devcenter.wercker.com/articles/werckeryml/).

We’ll start by creating a `wercker.yml` in the root of your repository. For now we’ll just specify the box. This will create a Ruby 1.9.3 environment (which is the default version) for us to work in. Make sure you indent your wercker.yml file correctly in the subsequent steps.

```yaml
# wercker.yml file
box: wercker/ruby
```

</br>

## Build

Since we’re going to be using [bundler](http://gembundler.com/) for our dependency management we need to include a Gemfile and a Gemfile.lock. Although a Gemfile.lock is optional, it’s recommended that you do include it in your repository (see for more information why you want to include your Gemfile.lock: [Clarifying the Roles of the .gemspec and Gemfile](http://yehudakatz.com/2010/12/16/clarifying-the-roles-of-the-gemspec-and-gemfile/)).

The first thing we’re going to do is install all our dependencies using `bundle install`. This will help us later on with packaging, and we’ll be able to use the gems to do certain tasks (such as testing). Wercker already has a bundle install step by default, which includes sensible defaults and it comes with caching. So we’ll add it to our wercker.yml.

```yaml
# wercker.yml file
box: wercker/ruby
build:
  steps:
    - bundle-install
```

</br>

If you want to do some custom action or run tests you can do so after the bundle-install step. For this tutorial we are focussing on packiging and deploying, so we haven’t included any other steps. See the [Ruby language guide](http://devcenter.wercker.com/articles/languages/ruby.html) on our dev center for more information on build steps such as `rake`.

The last step of our build is to package everything using the bundle package step. This step will copy all .gem files that are specified in the Gemfile.lock file and copy them in the ./vendor/cache folder. The next time bundle install will be used it will check the vendor/cache folder and use the files without downloading them from [rubygems.org](http://rubygems.org) or the git repositories. This can be very handy when rubygems.org is down or when a certain gem has been removed from rubygems.org. Also heavily firewalled servers don’t have to make any outbound connection (you do have to use bundle install using `--local`, to make sure it doesn’t check rubygems.org).

```yaml
# wercker.yml file
box: wercker/ruby
build:
  steps:
    - bundle-install
    # test steps, minify step, etc
    - bundle-package
```

</br>

The last step that wercker executes is `tar` the output and save it. So next time you start a deploy in the future, you can be certain that you’ll have all your gem files available.

## Deploy
For deployment we’re simply going to use the deployment framework [capistrano](https://github.com/capistrano/capistrano). We’ll start with a basic `deploy.rb` file (note: replace hostname with your server)

```ruby
# config/deploy.rb file
set :application, "sinatra-sample"
server "example.com", :app, :web
set :user, "ubuntu"
set :group, "ubuntu"
set :use_sudo, false
```
</br>
This deploy.rb simply contains one simple sinatra application and it sets some default values. For more information about these properties see the [capistrano documentation](https://github.com/capistrano/capistrano/wiki).

First we’re going to specify the source of the code. In most examples and quickstarts of capistrano you’ll see that the git repository is going to be used. This however is not desirable when using wercker, because there is a possibilty that build steps haved added, changed or removed certain files in the build output. To deploy the current working directory we use the following settings:

```ruby
# config/deploy.rb file
set :repository, "."
set :scm, :none
set :deploy_via, :copy
```
</br>
This will make capistrano compress the specified directory, *sftp* it to the server and expand it there.

Most ssh servers use a private key to do authentication. This key however is not available for wercker. So we need to make sure that wercker gets access to the key. We don’t want it to be included in our repository, so including it in our wercker.yml is out of the question. Wercker has the ability to add [environment variables](http://www.12factor.net/config) which are exposed to the deploy process.

![image](http://f.cl.ly/items/2k2o2m2009343v2I3A0P/Screen%20Shot%202013-06-07%20at%2012.20.16%20PM.png)

Create a WERCKER\_CAP\_PRIVATE_KEY environment variable for your deploy target and set it’s value to the private key which can access the server. We only have to write this environment variable to a file using a script step.

```yaml
# wercker.yml file
box: wercker/ruby
build:
  steps:
    - bundle-install
    # test steps, minify step, etc
    - bundle-package
deploy:
  steps:
    - bundle-install
    - script:
        name: write env var
        code: |-
          export CAP_PRIVATE_KEY=`mktemp`
          echo -e $WERCKER_CAP_PRIVATE_KEY > $CAP_PRIVATE_KEY
```
</br>
In our capistrano script we need to make sure that our private key gets used.

```ruby
# config/deploy.rb file
ssh_options[:keys] = [ENV["CAP_PRIVATE_KEY"]]
```
</br>

Finally we have to start the capistrano step, this can be done by including the cap step. The default arguments are good enough for us.

Below you can see the final `wercker.yml` file

```yaml
	# wercker.yml file
	box: wercker/ruby
	build:
	  steps:
	    - bundle-install
	    # test steps, minify step, etc
	    - bundle-package
	deploy:
	  steps:
	    - bundle-install
	    - script:
	        name: write env var
	        code: |-
	          export CAP_PRIVATE_KEY=`mktemp`
	          echo -e $WERCKER_CAP_PRIVATE_KEY > $CAP_PRIVATE_KEY
	    - cap
```
</br>

And we need to actually install the dependencies on our deployment server, this can simply be achieved by adding the bundler hooks to our capistrano script:

The final `deploy.rb` file for capistrano:


```ruby
	# config/deploy.rb file
	require 'bundler/capistrano'

	set :application, "sinatra-sample"
	server “example.com", :app, :web
	set :user, "ubuntu"
	set :group, "ubuntu"
	set :use_sudo, false

	set :repository, "."
	set :scm, :none
	set :deploy_via, :copy

	ssh_options[:keys] = [ENV["CAP_PRIVATE_KEY"]]
```
</br>
Although we demonstrated a simple deployment project, capistrano can do some really complex deployments. Starting these deploys from wercker can be really easy though.

We've setup a simple github repository were you can see all the code: [https://github.com/hatchan/sinatra-sample](https://github.com/hatchan/sinatra-sample).

More information about bundle install:

- [http://gembundler.com/v1.3/bundle_install.html](http://gembundler.com/v1.3/bundle_install.html)
- [http://gembundler.com/v1.3/man/bundle-install.1.html](http://gembundler.com/v1.3/man/bundle-install.1.html)

More information about bundle package:

- [http://gembundler.com/v1.3/bundle_package.html](http://gembundler.com/v1.3/bundle_package.html)
- [http://gembundler.com/v1.3/man/bundle-package.1.html](http://gembundler.com/v1.3/man/bundle-package.1.html)

More information about capistrano:

- [https://github.com/capistrano/capistrano](https://github.com/capistrano/capistrano)

And of course sign up for wercker [here](https://app.wercker.com/users/new) and check out our [dev center](http://devcenter.wercker.com) for more information!

Let us know how you are using wercker and capistrano for your deployments!