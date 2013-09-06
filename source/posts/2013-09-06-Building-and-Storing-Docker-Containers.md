---
title: Building and storing Docker Containers to S3
date: 2013-09-06
tags: docker,linuxcontainers,s3
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
<a href="http://docker.io">Docker</a> is an exciting technology which is
an abstraction layer on top of <a
href="http://lxc.sourceforge.net/">LXC</a> allowing for lightweight "OS-level"
virtualized machines that are extremely portable.
</h4>

At wercker, we're very excited about combining the power of
wercker's open delivery pipeline and the portability of Docker containers.

In the coming weeks we will post several tutorials, screencasts and
updates on our beta support for Docker containers.

<div class="text-center">
<a href="http://blog.wercker.com/2013/09/06/Building-and-Storing-Docker-Containers.html#form" class="button radius secondary">Apply for our beta Docker support</a>
</div>

![image](http://f.cl.ly/items/0B2m3d3k1N2S3p3b0g0B/wercker_loves_docker.png)

The short screencast below deals with building and storing a Docker container
from wercker to Amazon's S3. The container we will be building is a [RethinkDB](http://rethinkdb.com) database image.

READMORE

The Docker Index allows you to push and browse Docker images but it
could very well be that you do not want to store your image in public
environment or you're just want to play around with Docker and share
your image with others.

Watch the video below to see how to build and store your Docker image to S3.


<div class="flex-video">
<iframe src="//player.vimeo.com/video/73947111" width="500" height="313" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe> <p><a href="http://vimeo.com/73947111">Storing your Docker images to S3 with wercker</a> from <a href="http://vimeo.com/user19091499">Team wercker</a> on <a href="https://vimeo.com">Vimeo</a>.</p>
</div>

<a id="form"></a>
We've been hard at work at making delivery of Docker containers from wercker to various cloud providers, a reality.

Apply below for our Docker beta support and join us in making this happen.

Any additional awesome suggestions or feedback will be rewarded with some wercker stickers!

<div id="wufoo-z7x4m1">
Fill out my <a href="http://wercker.wufoo.com/forms/z7x4m1">online form</a>.
</div>
<script type="text/javascript">var z7x4m1;(function(d, t) {
var s = d.createElement(t), options = {
'userName':'wercker',
'formHash':'z7x4m1',
'autoResize':true,
'height':'679',
'async':true,
'header':'show'};
s.src = ('https:' == d.location.protocol ? 'https://' : 'http://') + 'wufoo.com/scripts/embed/form.js';
s.onload = s.onreadystatechange = function() {
var rs = this.readyState; if (rs) if (rs != 'complete') if (rs != 'loaded') return;
try { z7x4m1 = new WufooForm();z7x4m1.initialize(options);z7x4m1.display(); } catch (e) {}};
var scr = d.getElementsByTagName(t)[0], par = scr.parentNode; par.insertBefore(s, scr);
})(document, 'script');</script>


Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).
