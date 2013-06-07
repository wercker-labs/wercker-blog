---
title: Streamlining your Middleman deploys with wercker and S3
date: 2013-06-10
tags: middleman, wercker, s3, aws, deployment
author: Micha Hernandez van Leuffen
published: false
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
In a previous <a href="http://blog.wercker.com/2013/05/31/simplify-you-jekyll-publishing-process-with-wercker.html">post</a> we outlined how you can simplify the deployment process of your static Jekyll sites to Amazon S3 with wercker. This post goes into another static site generator called <a href="http://middlemanapp.com/">Middleman</a>.
</h4>

### Middlemanapp
We make heavy use of [Middleman](http://middlemanapp.com/) internally at wercker; our [dev center](http://devcenter.wercker.com) is built using Middleman and the [blog](http://blog.wercker.com) that you are currently reading as well, using the Middleman [blogging](http://middlemanapp.com/blogging/) [extension](https://github.com/middleman/middleman-blog).

In this post we will outline how you can streamline your Middleman deployment pipeline to Amazon Web Services S3.

### Prerequisites

* You have [created a free account](https://app.wercker.com/users/new/) at wercker.
* You have the code of your Middleman site hosted at [Github](http://github.com) or [Bitbucket](http://bitbucket.com).
* You have cloned a repository that contains a [Middleman](http://middlemanapp.com) site locally.
* You have an [Amazon S3 bucket](http://docs.aws.amazon.com/AmazonS3/latest/dev/HostingWebsiteOnS3Setup.html) that will serve your website.

### Creating our blog and adding it to wercker

As said we'll be using the Middleman blogging extension, so make sure you *gem install* it.

    middleman init --template blog

    git init

    git add .

    git commit -am 'init'

    git push origin master

</br>

Congrats, you have just created your blog with Middleman; now lets add it to wercker! Log into wercker and add your repository to wercker by clicking the 'add application button'. Select your repository from either GitHub or Bitbucket. Follow the steps given and make sure you give the `werckerbot` user, read rights on your repository at either Github or Bitbucket.


### Adding our S3 deploy target

We now need to specify a custom deploy target on wercker that we will use to setup the details for Amazon S3. Go to the settings tab for your application and under the section 'Deploy Target' click add 'Add deploy target'.

Here you can fill in the name for your deploy target (for instance 's3' or 'staging') and, if you prefer, select the [auto deploy option](http://blog.wercker.com/2013/06/05/Autodeployment.html) that allows you to automatically deploy specific branches (remember to auto deploy with care!).

Next you want to add the following [environment variables](www.12factor.net/config) that the build pipeline must leverage to sync your middleman app with S3. 

![image](http://f.cl.ly/items/1z3B0Y221P1i2M1u1f1q/Screen%20Shot%202013-06-07%20at%204.02.29%20PM.png)

Here you enter the details of your Amazon S3 bucket. The key and secret key can be found in the [AWS security credentials](https://portal.aws.amazon.com/gp/aws/securityCredentials) page.

### Creating our wercker.yml

Now it is time to define your build process. This is the pipeline that is run each time changes are pushed to the git repository.

Create a new file called `wercker.yml` in the root of your repository with the following content:

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
	        # Execute the heroku-deploy, heroku details can be edited
	        # online at http://app.wercker.com/
	        #- heroku-deploy
	
	        # Execute the s3sync deploy step, a step provided by wercker
	        - s3sync:
	            key_id: $AWS_ACCESS_KEY_ID
	            key_secret: $AWS_SECRET_ACCESS_KEY
	            bucket_url: $AWS_BUCKET_URL
	            source_dir: build/
```

The `s3sync` step synchronises a source directory with an Amazon S3 bucket. The `key_id`, `key_secret` and `bucket_url` options are set to the information from the deploy target, previously created on wercker. Only the `source` option is _hard configured_  to `build`. This is the default folder with the output from the `middleman build` command, which was previously executed in the build phase.

After you created the `wercker.yml` add it to your repository by executing the following commands in your terminal.

```bash
    git add wercker.yml
    git commit -m 'Add wercker.yml'
    git push origin master
```

This automatically triggers a new build on wercker as you can see below.

-------

<div class="authorCredits">
    <span class="profile-picture">
        <img src="https://secure.gravatar.com/avatar/d4b19718f9748779d7cf18c6303dc17f?d=identicon&s=192" alt="Micha Hernandez van Leuffen"/>
    </span>
    <ul class="authorCredits">

        <!-- author info -->
        <li class="authorCredits__name">
            <h4>Micha Hernandez van Leuffen</h4>
            <em>
                Micha is cofounder and CEO at wercker.
            </em>
        </li>

        <!-- info -->
        <li>
            <a href="http://beta.wercker.com" target="_blank">
                <i class="icon-company"></i> <em>wercker</em>
            </a>
            <a href="http://twitter.com/mies" target="_blank">
                <i class="icon-twitter"></i>
                <em> mies</em>
            </a>
        </li>

    </ul>
</div>

-------
##### last modified on: May 11, 2013
-------
