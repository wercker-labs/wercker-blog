---
title: Using wercker to publish to GitHub Pages
date: 2013-07-25
tags: opendelivery, github, jekyll
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
We previously wrote about deploying your <a href="http://blog.wercker.com/2013/06/10/Streamlining-Middleman-Deploys-to-s3.html">Middleman</a> and
<a
href="http://blog.wercker.com/2013/05/31/simplify-you-jekyll-publishing-process-with-wercker.html">Jekyll</a>
sites to <a href="http://aws.amazon.com/s3/">Amazon's S3</a> with wercker. In this post we want to outline how to deploy your static sites to <a href="http://pages.github.com/">GitHub Pages</a>
</h4>

![image](http://f.cl.ly/items/3z2K2z2g380M083g2140/wercker%2Bgitpages.png)

READMORE

You can leverage the [wercker pipeline](http://devcenter.wercker.com/articles/introduction/pipeline.html) to compile your Jekyll site and Sass assets. 
The [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) is used to set up this pipeline with build and deploy [steps](http://devcenter.wercker.com/articles/steps/).
This pipeline can be used in powerful ways and for a broad range of
purposes.

![image](http://f.cl.ly/items/2O3V2n3A1n2d3u3S363D/wercker_pipeline.png)

You can view a
[wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) for
building a [Jekyll](http://jekyllrb.com/) site below:

``` yaml
box: wercker/ruby
build:
  steps:
    # Install dependencies
    - bundle-install
    
    # Execute jeykyll doctor command to validate the 
    # site against a list of known issues.
    - script:
        name: jekyll doctor
        code: bundle exec jekyll doctor
    - script:
        name: sass compile
        code: bundle exec sass --style compressed assets/scss/styles.scss:assets/css/styles.min.css --debug-info
    
    # Generate staging, no drafts in here
    - script:
        name: generate staging site
        code: |-
          bundle exec jekyll build --trace --destination ./_staging
          ls -l ./_staging
    # Do not allow indexing on staging
    - create-file:
        name: generate staging robots.txt
        filename: ./_staging/robots.txt
        content: |-
          # Go away! This is a staging version
          # of born2code.net, no need to index it.
          User-agent: *
          Disallow: /
    - script:
        name: generate production site
        code: |-
          bundle exec jekyll build --trace --destination ./_production
          ls -l ./_production
    - create-file:
        name: generate production robots.txt
        filename: ./_production/robots.txt
        content: |-
          User-agent: *
          Allow: /
          Sitemap: http://born2code.net/sitemap.xml
```

It consists of various steps to build my pages, do some Sass compilation
and even generate both a production and staging build of my site.

## Adding our Deploy Target to wercker

Instead of deploying to S3, we are now going to deploy to [GitHub
pages](http://pages.github.com). 

First, we create a personal access token on GitHub that we will use in
our git remote information. We can do this in the **Application**
section in our GitHub settings:

![image](http://f.cl.ly/items/0L2J03450X340u0I190w/create-auth-key.png)

**NOTE:** this token has already been deleted since writing this post

Next, we create a deploy target on wercker for GitHub Pages. Let's call
it **production**. We also add an [environment variable](http://12factor.net/config) that we hide
from our log, called **GIT_REMOTE** holding our newly created token in
the GitHub remote url:

![image](http://f.cl.ly/items/2z353n3K0C2W3C1V2Y3d/add-to-deploy-target.png)

The git remote url pattern is: `https://{TOKEN}@github.com/{ACCOUNT}/{REPOSITORY}.git`

## Updating our wercker.yml

Now we are ready to update our
[wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) with
the deployment information that we just added.

For this we use a custom [deploy
step](http://devcenter.wercker.com/articles/introduction/deploys.html)
in our **wercker.yml** file:

``` yaml
deploy :
  steps :
    - script:
        name: Configure git
        code: |-
          git config --global user.email "pleasemailus@wercker.com"
          git config --global user.name "wercker"
          
          # remove current .git folder
          rm -rf .git
    - script:
        name: Deploy to github pages
        code: |-
          git init
          git add .
          git commit -m "deploy commit from $WERCKER_STARTED_BY"
          git push -f $GIT_REMOTE master:gh-pages
```

In this step we supply our git configuration as we use it on GitHub and
remove the current **.git** folder which is part of the build artifact
on wercker.

Finally we create a new repo in our box that we will deploy to the
**gh-pages** branch and leveraging our `$GIT_REMOTE` environment variable
that we added in the previous step.

Then we push it out to GitHub Pages!

There is definitely some room for improvement to create a more elegant
solution, for instance through a [custom deploy step](http://blog.wercker.com/2013/07/23/Spotlight-on-pipeline-steps.html) specifically for
GitHub pages.

This tutorial however should get you started though and don't forget to
tell us about the improvements you have in mind.

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.





