---
title: Deploying your PHP application to fortrabbit
date: 2013-08-28
tags: deployment, golang, AppEngine
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
---

Today we got a support ticket from a user that wants to deploy his PHP application to [fortrabbit](http://fortrabbit.com/ "fortrabbit homepage"), a PHP as a service provider. To help this customer I decided to create a small sample app and try to deploy it to fortrabbit. This blogpost describes the steps I took. I will give you a short summery right away: It was very easy!

## The PHP application

I have a very simple PHP application that echo's some cities in json format from the [index.php](https://github.com/pjvds/php-fortrabbit-sample-app/blob/master/index.php). Here is the content of [index.php](https://github.com/pjvds/php-fortrabbit-sample-app/blob/master/index.php):

``` php
<?php
$cities = array("Amsterdam", "San Francisco", "Berlin",
                "New York", "Tokyo", "London");

header('Content-Type: application/json');
echo json_encode($cities, JSON_PRETTY_PRINT);
?>
```

It also contains some [tests](https://github.com/pjvds/php-fortrabbit-sample-app/blob/master/tests/) and has a small [wercker.yml](https://github.com/pjvds/php-fortrabbit-sample-app/blob/master/wercker.yml) that defines the build pipeline. Here is the content of [wercker.yml](https://github.com/pjvds/php-fortrabbit-sample-app/blob/master/index.php):

``` yaml
# Execute the pipeline with the wercker/php box
box: wercker/php
build:
  steps:
    # Install dependencies with composer
    - script:
        name: install dependencies
        code: |-
            composer install --no-interaction
    # Spin webservice and serve site
    - script:
        name: Serve application
        code: php -S localhost:8000 >> /dev/null &
    # Execute integration tests with php unit
    - script:
        name: PHPUnit integration tests
        code: phpunit --configuration phpunit.xml
```

## Generating SSH Key

Wercker has the ability to generate SSH keys that are available from the build and deployment pipeline. These can be generated from the application settings tab at wercker. I generate a key with the name `fortrabbit` and wercker shows me the public key.

![generating an ssh key at wercker.com](/images/posts/deploying-your-php-application-to-fortrabbit/generating-ssh-key.png)

## Adding public key to fortrabbit

To be able to git push deploy to fortrabbit we need to add the public key part of the SSH key pair to our application. This can be done in the manage view of your application via the [application overview](https://my.fortrabbit.com/apps).

![application overview](/images/posts/deploying-your-php-application-to-fortrabbit/app-overview.png)

Navigate to the git tab.

![git tab](/images/posts/deploying-your-php-application-to-fortrabbit/git-tab.png)

Enter a name for the git user, I picked wercker and paste the public key part into the big text area.

![git tab](/images/posts/deploying-your-php-application-to-fortrabbit/add-key.png)

Hit the `Save Git Users` button to confirm.

## Deployment pipeline

I have played a bit and came up with the following deployment pipeline that I added to the [wercker.yml](https://github.com/pjvds/php-fortrabbit-sample-app/blob/738f0b69d403b6045f809470bc5b27b97a9f58db/wercker.yml#L18-L56):

``` yaml
deploy:
  steps:
    - add-to-known_hosts:
        hostname: $FORTRABBIT_GIT_HOST
    - script:
        name: Setup git repository
        code: |-
          # Remove existing git repository if exists
          if [ -d ".git" ]; then rm -rf .git; fi

          # Configure git user
          git config --global user.name "wercker"
          git config --global user.email pleasemailus@wercker.com

          # Initialize new repository and add everything
          git init
          git add .
          git commit -m 'Deploy commit'

          # Add fortrabbit remote
          git remote add fortrabbit "$FORTRABBIT_GIT_REMOTE"
    - script:
        name: Make .SSH directory
        code: mkdir -p "$HOME/.ssh"
    - create-file:
        name: Write SSH key
        filename: $HOME/.ssh/id_rsa
        overwrite: true
        content: $FORTRABBIT_KEY_PRIVATE
    - script:
        name: Set permissions for SSH key
        code: |-
          chmod 0400 $HOME/.ssh/id_rsa
    - script:
        name: Git push deploy
        code: |-
          git push fortrabbit master -f
```

Most steps should be self explaining, but here is what is it does in plain English:

  * Trust forstrabbit git hostname, eq: `it2.eu1.frbit.com`
  * Setup git repository and add all files to push
  * Add SSH private key for authentication
  * Git push deploy to fortrabbit

## Add deploy target

The last step is to add an deploy target to the application at wercker. This can be done via the settings tab of the application:

![add deploy target](/images/posts/deploying-your-php-application-to-fortrabbit/add-deploy-target.png)

I name it production, and enable auto deploy for the master branch.

![deploy target basic properties](/images/posts/deploying-your-php-application-to-fortrabbit/deploy-target-basic-properties.png)

We can add environment variables to the deploy target to make information available during the deployment. I start by exposing the SSH key that I have added earlier:

![add ssh key variable](/images/posts/deploying-your-php-application-to-fortrabbit/add-key-var.png)

Next I add the two text variables `FORSTRABBIT_GIT_REMOTE` and `FORSTRABBIT_GIT_HOST`, which I set to the values displayed on the git tab at fortrabbit.

## Deploy!

Navigate to your latest build and choose deploy:

![deploy](/images/posts/deploying-your-php-application-to-fortrabbit/deploy.png)

When the deploy is finish I proudly look at the running version of the application:

![cities running at fortrabbit](/images/posts/deploying-your-php-application-to-fortrabbit/cities-running-at-fortrabbit.png)

Source: [cities.eu1.frbit.net](http://cities.eu1.frbit.net/)
