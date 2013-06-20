---
title: Introducing our new add application flow
date: 2013-06-19
tags: wercker, ui, design, flow
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
published: false
---

<h4 class="subheader">
At wercker, we love getting feedback from our users as it helps us finetuning
and improving the product in a meaningful way for our users. In this post we're
introducing the new <strong>add application flow</strong> that we just
deployed to production.

Sign up for wercker <a href="https://app.wercker.com/users/new/">for free here</a>

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

![image]()

We now present you with a list of repositories and also show if a repository is
private, public and a fork of another repo.

![image]()

Next, we ask you to add werckerbot as a collaborator to your repository on either GitHub or Bitbucket. This is necessary as 
wercker needs to be able to **temporary** clone your repository, in order to run your tests and deploy your application.
You can read more on **werckerbot** at our [dev center](http://devcenter.wercker.com/articles/gettingstarted/werckerbot.html)

![image]()

Now wercker is ready to help you with the `wercker.yml` file. It could of course be the case that you've already 
added a `wercker.yml` to your repository.

![image]()

Done!

We think this new flow is a great improvement from our previous version, with
concise information but still allowing you to quickly add an application to
wercker. Even fore more experienced users you are still able to swiftly get your project into wercker and get started.

It also helps you in creating your build and deploy pipeline using the
[wercker.yml(http://devcenter.wercker.com/articles/werckeryml/) file, with sensible
defaults in place.

Let us know what you think, as we always welcome more feedback.

As always, signing up for wercker is [free and
easy]((https://app.wercker.com/users/new/).

