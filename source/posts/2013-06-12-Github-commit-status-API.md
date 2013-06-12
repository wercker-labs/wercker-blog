---
title: Attaching build statuses to commits on Github
date: 2013-06-12
tags: wercker, github, git, pullrequests, api
author: Micha Hernandez van Leuffen
published: false
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
At wercker we make heavy use of GitHub and are big fans of what Tom, Chris, PJ, Scott and the rest of the GitHub team have built over the last years. When we started wercker, we wanted to integrate with GitHub to make the user experience for our continuous delivery smooth and seamless but also have a strong focus on collaboration. In this post we go into wercker's integration with the <a href="https://github.com/blog/1227-commit-status-api">GitHub commit status api</a> for pull requests.
</h4>

READMORE

A while back GitHub announced [commit status api](https://github.com/blog/1227-commit-status-api) and wercker supports attaching the build status of your commit in two ways. In both cases [webhooks](https://help.github.com/articles/post-receive-hooks) are triggered each time you do a `git push` to your repository. These webhooks communicate with and tell us that a build should be started from your latest push.

### Scenario 1: Status on Commit

Even when pushing to your own repositories and without utilizing pull requests, results of these builds are attached to your commit.

Below you can see that the latest build for this blog has successfully completed on wercker.

![image](http://f.cl.ly/items/1X3F270u1Y3H2c413j3Q/Screen%20Shot%202013-06-12%20at%201.34.13%20PM.png)

If we view the branch of this commit on GitHub you can see that hovering over the &#10003;checkmark will showcase that the build finished with a success status. Clicking that &#10003;checkmark will take you to the [build page on wercker](https://app.wercker.com/#build/51b84324345a2a453d002cda) and give you more details on the various buildsteps, such as unittests, bundle install and [middleman processing](http://middlemanapp.com) the build went through.

![image](http://f.cl.ly/items/012I2h3x0A1b2E0C0U2G/Screen%20Shot%202013-06-12%20at%201.41.28%20PM.png)

### Scenario 2: Pull Requests

The second scenario is the support for [pull
requests](https://help.github.com/articles/using-pull-requests). Pull
requests are an awesome way of collaborating on projects: you fork an
existing repository, `git push` your improvements to your forked repo
and do a pull request on the original repository to contribute back your
code. When you do a pull requests, wercker again gets notified and
starts building your latest commit.

![image](http://f.cl.ly/items/2p17330V153v163W3d3G/Screen%20Shot%202013-06-12%20at%202.11.28%20PM.png)
