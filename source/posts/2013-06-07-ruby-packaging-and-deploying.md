---
title: Packing and deploying Ruby apps
date: 2013-06-07
tags: ruby, bundler, capistrano, deployment
author: Benno van den Berg
gravatarhash: dff7a3e4eadab56aa69a24569cb61e98
published: false
---
<h4 class="subheader">
In this tutorial we’re going to show you the recommended way to package and deploy Ruby applications with wercker, bundler and capistrano.
</h4>

You should have some knowledge of wercker and you should have a application in wercker.

We’ll start by creating a wercker.yml in the root of your repository. For now we’ll just specify the box. This will create a Ruby 1.9.3 environment for us to work in.

```yaml
box: wercker/ruby
```

## Build ##

Since we’re going to be using bundler for our dependency management we need to include a Gemfile and a Gemfile.lock. Although a Gemfile.lock is optional, it’s recommended that you do include it in your repository (more information why you want to include your Gemfile.lock: [Clarifying the Roles of the .gemspec and Gemfile](http://yehudakatz.com/2010/12/16/clarifying-the-roles-of-the-gemspec-and-gemfile/)).

The first thing we’re going to do is install all our dependencies using bundle install. This will help us later on with packaging and we’ll be able to use the gems to do certain tasks (such as testing). Wercker has already created a bundle install step, which comes with sane defaults and it comes with caching. So we’ll add it to our wercker.yml.

```yaml
box: wercker/ruby
build:
  steps:
    - bundle-install
```

If you want to do some custom action or run tests you can do so after the bundle-install step. For this tutorial we are focussing on packiging and deploying, so haven’t included any other steps.

The last step of our build is to package everything using the bundle package step. This step will copy all .gem files that are specified in the gemfile.lock file and copy them in the ./vendor/cache folder. The next time bundle install will be used it will check the vendor/cache folder and use the files without downloading them from rubygems.org or the git repositories. This can be very handy when rubygems.org is down or when a certain gem has been removed from rubygems.org. Also heavily firewalled server don’t have to make any outbound connection (you do have to use bundle install using --local, to make sure it doesn’t check rubygems.org).

```yaml
box: wercker/ruby
build:
  steps:
    - bundle-install
    # test steps, minify step, etc
    - bundle-package
```

The last step which wercker does is tar the output and save it. So next time you start a deploy in the future, you can be certain that you’ll have all your gem files available.

## Deploying ##
For deployment we’re simply going to use the deployment framework [capistrano](https://github.com/capistrano/capistrano). We’ll start with a basic deploy.rb file (note: replace hostname with your server)

```ruby
set :application, "sinatra-sample"
server "example.com", :app, :web
set :user, "ubuntu"
set :group, "ubuntu"
set :use_sudo, false
```

This deploy.rb simply contains 1 simple application and it sets some default values. For more information about these properties see the [capistrano documentation](https://github.com/capistrano/capistrano/wiki).

First we’re going to specify the source of the code. In most examples and quickstarts of capistrano you’ll see that the git repository is going to be used. This however is not desirable when using wercker, because some build steps added, changed or removed certain files in the build output. To deploy the current working directory we use the following settings.

```ruby
set :repository, "."
set :scm, :none
set :deploy_via, :copy
```

This will make capistrano compress the specified directory, sftp it to the server and expand it there.

Most ssh servers use a private key to do authentication. This key however is not available for wercker. So we need to make sure that wercker gets access to the key. We don’t want it to be included in our repository, so including it in our wercker.yml is out of the question. Wercker has the ability to add environment variables which are exposed to the deploy process. Create a WERCKER\_CAP\_PRIVATE_KEY environment variable for your deploy target and set it’s value to the private key which can access the server. We only have to write this environment variable to a file using a script step.

```yaml
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

In our capistrano script we need to make sure that our private key gets used.

```ruby
ssh_options[:keys] = [ENV["CAP_PRIVATE_KEY"]]
```

Finally we have to start the capistrano step, this can be done by including the cap step. The default arguments are good enough for us.

Final wercker.yml

```yaml
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

And we need to actually install the dependencies on our deployment server, this can simply be achieved by adding the bundler hooks to our capistrano script:

Final deploy.rb

```ruby
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