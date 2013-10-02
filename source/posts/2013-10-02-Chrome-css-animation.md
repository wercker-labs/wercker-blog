---
title: CSS animations in Chrome
date: 2013-10-02
tags: chrome css animation
author: Lindsey Bateman
gravatarhash: e1c82876f21cdafafd2b01a1e625f587
---
<h4 class="subheader">
An CSS animation bug causing 100% CPU in Chrome
</h4>

We added several animations to enhance the experience of our tool.
Like a spinning icon when a build is in progress or a fade in when a new feed item appears.
Fairly simple CSS animations to make wercker come to life and give it more an application feel.

All fun until an user gave u much appreciated feedback that ...
READMORE

Wercker was taking up a 100% of CPU usage in Chrome.
We discovered this was specifically a feature of Chrome, we don't know in which version this started to be a problem.
But probably it appeared in one of the latests versions of Chrome, my current Chrome version is 29.0.1547.76.

![CSS animations in Chrome](/images/posts/chromecssanimations/chrometaskmanager.png)

So what's going on and what is causing it… At first we couldn't reproduce it or by accident and we had a blowing MacBook pro.
With multiple tabs open and at least one being wercker running in the background the 100% CPU bug appeared.
When focussing on the wercker tab the 100% CPU immediately stopped and dropped to a normal level, hmm what the hell.

![Overheating MacBook pro](/images/posts/chromecssanimations/heatmap.jpg)

After some research it seemed to be CSS animations were causing this problem.
We commented out all the CSS animations so we could then add them again one by one.

Our sidebar menu animates in after signing in to wercker to enhance the feeling the sidebar is something linked to you as a user and only when logged-in.
We use **animation keyframes** and the `-webkit-animation-fill-mode: forwards;` so that the sidebar remains in its position of the last animation keyframe.
Normally when the animation is done the CSS property will snap to its original setting.

<style type="text/css">
		.blue__block {
			width: 200px;
			height: 200px;
			background-color: blue;
			margin-right: auto;
			margin-left: auto;
		}

		.blue__block.active {
			-webkit-animation: rotating 1s linear;
			-webkit-animation-fill-mode: forwards;
		}

		.red__block {
			width: 100px;
			height: 100px;
			background-color: red;
			-webkit-animation: rotating 4s linear;
			-webkit-animation-fill-mode: forwards;
		}

		@-webkit-keyframes rotating {
		     from{
		         -webkit-transform: rotate(0deg);
		     }
		     to{
		         -webkit-transform: rotate(45deg);
		    }
		}

	</style>

 <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>

<script type="text/javascript">
	var toggleBlueBox = function(){
		$(".blue__block").toggleClass("active");
	};
	window.setInterval(toggleBlueBox, 5000);
</script>

<div>
	<div class="blue__block">
		<div class="red__block"></div>
	</div>
</div>

So this is great for us but not for Chrome, the CSS animations using the forwards fill-mode are causing the 100% CPU bug.
To be more specific it only occurs when the CSS animation ends and when the tab is inactive. We got it!

No we didn't so there is one aspect more to this. It only occurs when the CSS animation is triggered when you're not looking.
In our case we have a feed with messages about your builds, deploys and team updates.
When a new message appears it gets added with Javascript and nicely appears at the top of the feed with a fadein. Now the 100% CPU big appears!


## Earn some stickers of your own!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Thanks for tuning in, and there will be more posts next week! Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.


