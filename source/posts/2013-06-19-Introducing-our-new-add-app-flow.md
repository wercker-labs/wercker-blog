---
title: Introducing our new add application flow
date: 2013-06-19
tags: wercker, ui, design, flow
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
At wercker, we love and appreciate receiving feedback from our users and the developer community as it helps us finetune
and improve wercker in a meaningful way. In this post we're
introducing the new <strong>add application flow</strong> that we just
deployed to production and which we couldn't have delivered without this feedback.

As always you can sign up for wercker <a href="https://app.wercker.com/users/new/">for free here</a>

</h4>

READMORE

The initial reactions to our **flow** of adding an application on wercker were
that it was very quick to do so, but did not point you towards setting up your
actual [build](http://devcenter.wercker.com/articles/introduction/builds.html)
and [deployment](http://devcenter.wercker.com/articles/introduction/deployment.html) pipeline.

## wercker.yml

The [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file is a readable and concise DSL (in [yaml](http://www.yaml.org/) that allows
you to set up this pipeline and the environment it should run in. A small example of such a pipeline is the following default `wercker.yml`
for Ruby applications:

```yaml
box: wercker/ruby
build:
  steps:
    - bundle-install
    - script:
        name: rake
        code: bundle exec rake
```
In our new workflow for adding an application to wercker, we wanted to showcase the power of the `wercker.yml` DSL but
not be intimidating.

## The new add application flow

Our new workflow consists of small steps that guide you towards setting up
your application on wercker but also points you in the right direction of
creating your pipeline using the `wercker.yml` file.

We first allow you to select your git provider, currently either
[GitHub](http://github.com) or [Bitbucket](http://bitbucket.org). If you've
already authorized wercker to connect with either of these platforms, we will
show this as well.

In the case of GitHub, you can even choose between private and public
repositories (or both).

![image](http://f.cl.ly/items/0v080B3l1b1I2P3P3x2z/Screen%20Shot%202013-06-21%20at%2011.53.17%20AM.png)

We now present you with a list of repositories and also show if a repository is
private, public and a fork of another repo. We introduced a search box which you use to filter your application,
making selection even snappier.

![image](http://f.cl.ly/items/0Z1c0A1k0g1c0j2h082z/Screen%20Shot%202013-06-21%20at%2011.53.43%20AM.png)

Next, we ask you to add werckerbot as a collaborator to your repository on either GitHub or Bitbucket. This is necessary as
wercker needs to be able to **temporary** clone your repository, in order to run your tests and deploy your application.
You can read more on **werckerbot** at our [dev center](http://devcenter.wercker.com/articles/gettingstarted/werckerbot.html)

![image](http://f.cl.ly/items/2J3U202n06120u0G1i0a/Screen%20Shot%202013-06-21%20at%2011.53.58%20AM.png)

Now, wercker is ready to help you with the `wercker.yml` file. It could of course be the case that you've already
added a `wercker.yml` to your repository. The new flow automatically detects the presence of a `wercker.yml` in your repository.

If the `wercker.yml` is not present we will try to detect the programming language of your project and present a suggested `wercker.yml` with sensible defaults.

![image](http://f.cl.ly/items/3V33302R3W1F3z03461m/Screen%20Shot%202013-06-21%20at%2012.00.50%20PM.png).

Again, you can read more on the `wercker.yml` file and its possibilities at our [dev center](http://devcenter.wercker.com).


Finally when you're done we present you with a success message and allow you to make your project **public** on wercker, which is great for open source projects.

We think this new flow is a great improvement from our previous version, with
concise information yet still allowing you to quickly add an application to
wercker, even for experienced users.

It also assists you in creating your build and deploy pipeline using the
[wercker.yml(http://devcenter.wercker.com/articles/werckeryml/) file, with sensible
defaults in place.

Let us know what you think, as we always welcome more feedback!

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and
easy]((https://app.wercker.com/users/new/).

