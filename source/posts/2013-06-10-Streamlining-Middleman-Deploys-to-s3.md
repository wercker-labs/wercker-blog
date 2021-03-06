---
title: Streamlining your Middleman deploys with wercker and S3
date: 2013-06-10
tags: middleman, wercker, s3, aws, deployment
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
In a previous <a href="http://blog.wercker.com/2013/05/31/simplify-you-jekyll-publishing-process-with-wercker.html">post</a> we outlined how you can simplify the deployment process of your static Jekyll sites to Amazon S3 with wercker. This post goes into another static site generator called <a href="http://middlemanapp.com/">Middleman</a>.
</h4>

READMORE

### Middlemanapp
We make heavy use of [Middleman](http://middlemanapp.com/) internally at wercker; our [dev center](http://devcenter.wercker.com) is built using Middleman and the [blog](http://blog.wercker.com) that you are currently reading as well, using the Middleman [blogging](http://middlemanapp.com/blogging/) [extension](https://github.com/middleman/middleman-blog).

In this post we will outline how you can streamline your Middleman deployment pipeline to Amazon Web Services S3. This article is also available on our [dev center](http://devcenter.wercker.com/articles/deployment/middleman.html)

### Prerequisites

* You have [created a free account](https://app.wercker.com/users/new/) at wercker.
* You have the code of your Middleman site hosted at [Github](http://github.com) or [Bitbucket](http://bitbucket.com).
* You have cloned a repository that contains a [Middleman](http://middlemanapp.com) site locally.
* You have an [Amazon S3 bucket](http://docs.aws.amazon.com/AmazonS3/latest/dev/HostingWebsiteOnS3Setup.html) that will serve your website.

### Creating our blog and adding it to wercker

As said we'll be using the Middleman blogging extension, so make sure you *gem install* it.

```bash
middleman init --template blog

git init

git add .

git commit -am 'init'

git push origin master
```

Congrats, you have just created your blog with Middleman; now lets add it to wercker! Log into wercker and add your repository to wercker by clicking the 'add application button'. Select your repository from either GitHub or Bitbucket. Follow the steps given and make sure you give the `werckerbot` user, read rights on your repository at either Github or Bitbucket.


### Adding our S3 deploy target

We now need to specify a custom deploy target on wercker that we will use to setup the details for Amazon S3. Go to the settings tab for your application and under the section 'Deploy Target' click add 'Add deploy target'.

Here you can fill in the name for your deploy target (for instance 'S3' or 'staging') and, if you prefer, select the [auto deploy option](http://blog.wercker.com/2013/06/05/Autodeployment.html) that allows you to automatically deploy specific branches (remember to auto deploy with care!).

Next you want to add the following [environment variables](http://12factor.net/config) that the build pipeline must leverage to sync your middleman app with S3.

![image](http://f.cl.ly/items/1z3B0Y221P1i2M1u1f1q/Screen%20Shot%202013-06-07%20at%204.02.29%20PM.png)

Here you enter the details of your Amazon S3 bucket. The key and secret key can be found in the [AWS security credentials](https://portal.aws.amazon.com/gp/aws/securityCredentials) page.

### Creating our wercker.yml

Now it is time to define your build process. This is the pipeline that is run each time changes are pushed to the git repository.

Create a new file called `wercker.yml` in the root of your repository with the following content (please make sure you indent the wercker.yml file correctly):

```yaml
box: wercker/ruby
build:
    steps:
        # Execute the bundle install step, a step provided by wercker
        - bundle-install
        # Execute a custom script step.
        - script:
            name: middleman build
            code: bundle exec middleman build --verbose
deploy:
    steps:
        # Execute the s3sync deploy step, a step provided by wercker
        - s3sync:
            key_id: $AWS_ACCESS_KEY_ID
            key_secret: $AWS_SECRET_ACCESS_KEY
            bucket_url: $AWS_BUCKET_URL
            source_dir: build/
```

The `s3sync` step synchronises a source directory with an Amazon S3 bucket. The `key_id`, `key_secret` and `bucket_url` options are set to the information from the deploy target, previously created on wercker. Only the `source` option is _hard configured_  to *build/* folder. This is the default folder with the output from the `middleman build` command, which was previously executed in the build phase.

After you've created the `wercker.yml` add it to your repository by executing the following commands in your terminal.

```bash
git add wercker.yml
git commit -m 'Add wercker.yml'
git push origin master
```
</br>

This automatically triggers a new build on wercker as you can see below.

![image](http://f.cl.ly/items/3z2N3k1B1E1l2C1V0B0j/Screen%20Shot%202013-06-07%20at%204.24.46%20PM.png)

If everything went well you are now ready to deploy this green build (if you didn't activate auto deploy of course!) to the S3 bucket that you have defined as a deploy target in a previous step. Select your build and click on the 'Deploy this build' button.

![image](http://f.cl.ly/items/3q2h0M333o0k2K3l2P2a/Screen%20Shot%202013-06-07%20at%204.35.06%20PM.png)

Congratulations your blog built with middleman and deployed via wercker is now live on S3!

Tweet out your green build and we'll send you some wercker stickers! (don't forget to @reply us so we notice)

Let us know if you have any feedback on creating and deploying your middleman apps with wercker.
