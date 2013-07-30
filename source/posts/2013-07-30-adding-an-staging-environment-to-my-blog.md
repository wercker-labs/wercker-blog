---
title: Adding an staging environment to jekyll blog
date: 2013-07-30
tags: jekyll, s3
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
published: false
---

I am using [jekyll](http://jekyllrb.com) for my [blog](http://born2code.net) and host it on [Amazon S3](http://aws.amazon.com/s3/). I use [wercker](http://wercker.com) to automate the content generation and deployment pipeline. In this post I want to outline how I introduced an staging environment next to my production environment.

## Staging

Your first question may be why I want to add a staging area for my blog in the first place? This allows me to test CSS hacks or other fixes for my site. I also use it for collaboration and proof reading.

## Delivery pipeline

On every git push wercker will execute the build pipeline which I defined in the [wercker.yml](https://github.com/pjvds/born2code.net/blob/master/wercker.yml) file. You can learn more about this file at [wercker devcenter](http://devcenter.wercker.com/articles/werckeryml/). Here I will go through the pipeline steps one by one.

### Build phase

The build pipeline consists of the following steps:

#### Bundle install

    # Install dependencies
    - bundle-install

Gets the gems like [Jekyll](http://jekyllrb.com/), [Redcarpet](https://github.com/vmg/redcarpet) and [sass](http://sass-lang.com/). I used the [bundle-install](https://app.wercker.com/#applications/51c829d13179be44780020be/tab/details) from wercker which does some smart things with caching to reduse execution time.

#### Jekyll doctor

    # Execute jeykyll doctor command to validate the
    # site against a list of known issues.
    - script:
        name: jekyll doctor
        code: bundle exec jekyll doctor

Validates the content against deprecations.

#### Sass compile

	- script:
        name: sass compile
        code: bundle exec sass --style compressed assets/scss/styles.scss:assets/css/styles.min.css --debug-info

Compiles the [sassy css files](https://github.com/pjvds/born2code.net/tree/master/assets/scss) and minifies the outcome.

#### Jekyll build

    # Generate staging, no drafts in here
    - script:
        name: generate staging site
        code: |-
          bundle exec jekyll build --trace

Builds the website into `_site`.

#### Copy site output

    - script:
        name: copy site output
        code: |-
          # Copy site to output directory for staging and production
          mkdir -P "$WERCKER_OUTPUT_DIR/{staging,production}
          cp --recursive _site/* "$WERCKER_OUTPUT_DIR/staging"
          cp --recursive _site/* "$WERCKER_OUTPUT_DIR/production"

Copies the `jekyll build` result from `_site` to staging and production directories in the output directory of the build pipeline. This directory is the input for the deployment pipeline and will contain both version of the site.

#### Generate staging robots.txt

	- create-file:
        name: generate staging robots.txt
        filename: $WERCKER_OUTPUT_DIR/staging/robots.txt
        content: |-
          # Go away! This is a staging version
          # of born2code.net, no need to index it.
          User-agent: *
          Disallow: /

Create a robots.txt file for the staging version of the website, which specifies to not index anything.

#### Generate production robots.txt

    - create-file:
        name: generate production robots.txt
        filename: $WERCKER_OUTPUT_DIR/production/robots.txt
        content: |-
          User-agent: *
          Allow: /
          Sitemap: http://born2code.net/sitemap.xml

Create a robots.txt file for the production version of the website, which specifies to index anything everything and hints to the sitemap.xml.

### Deploy phase

The deploy pipeline consists of the following step:

#### s3sync

    - s3sync:
        key-id: $KEY
        key-secret: $SECRET
        bucket-url: $BUCKET
        source-dir: $SOURCE

Synchronized the directory `$SOURCE` (_staging_ or _production_) to the `$BUCKET` (_s3://staging.born2code.net/_ or _s3://born2code.net/_) bucket on Amazon s3. The values of these variables are specified in by the deploy target.

## Configure the deploy targets

In wercker I add two deploy targets which I name staging and production. Which will hold the configuration values for the deploy pipeline.

![deploy targets](http://f.cl.ly/items/3i2G303h363G3g33323y/targets.png)

### Specify deployment options

I enable auto deployment for both targets. All pushes to the development branch will trigger an deploy to staging, and pushes the master branch will trigger an deployment to production. For both I specify the variables are follow:

![target config](http://f.cl.ly/items/1V2L0Q3j3v3u1p2a3b2K/target-config.png)

_note: this are not my real keys!_

## Done

Now the following will trigger an deployment to staging:

	$ git checkout development
	$ vim _posts/2013-07-30-adding-an-staging-environment-to-my-blog.md
	$ git add .
	$ git commit -m 'Adds post: adding staging environment to my blog'
	$ git push

I can review my blog at staging: [staging.born2code.net](http://staging.born2code.net) and when I think it is all good to go to production, I just merge with master and push.

	$ git checkout master
	$ git merge development
	$ git push
