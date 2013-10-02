---
title: CSS animations in Chrome
date: 2013-10-02
tags: chrome css animation
author: Lindsey Bateman
gravatarhash: e1c82876f21cdafafd2b01a1e625f587
published: false
---
<h4 class="subheader">
A CSS animation bug causing 100% CPU in Chrome
</h4>

We added several animations to enhance the experience of our tool.
Like a spinning icon when a build is in progress or a fade in when a new feed item appears.
Fairly simple CSS animations to make wercker come to life and give it more an application feel.

All fun until an user gave u much appreciated feedback that ...
READMORE

Wercker was taking up a 100% of CPU usage in Chrome.
We discovered this was specifically a feature of Chrome, we don't know in which version this started to be a problem.
But probably it appeared in one of the latests versions of Chrome, my current Chrome version is 30.0.1599.66 (released today).

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

So this is great for us but not for Chrome, the CSS animations using `-webkit-animation-fill-mode: forwards;` are causing the 100% CPU bug.
To be more specific it only occurs when the CSS animation ends and when the tab is inactive.



####Try this at home


<style type="text/css">

	.demo {
		margin: 40px;
	}

	.red__block {
		width: 50px;
		height: 50px;
		border: 5px solid red;
		border-color: red red transparent transparent;
		border-radius: 25px;
		background-color: transparent;
	}

	.red__block.active {
		-webkit-animation: rotating 3s linear;
		-webkit-animation-fill-mode: forwards;
		-moz-animation: rotating 3s linear;
		animation-fill-mode: forwards;
	}

	@-webkit-keyframes rotating {
	     from{
	         -webkit-transform: rotate(0deg);
	     }
	     to{
	         -webkit-transform: rotate(1440deg);
	    }
	}

	@-moz-keyframes rotating {
	     from{
	         -moz-transform: rotate(0deg);
	     }
	     to{
	         -moz-transform: rotate(1440deg);
	    }
	}

</style>
<a class="js-toggle-play button">Toggle animation</a> and switch to an other or a new tab.

<div class="demo">
	<div class="red__block"></div>
</div>

<script type="text/javascript">
	var toggleBlueBox = function(){
		$('.red__block').toggleClass("active");
	};
	$(".js-toggle-play").on("click", toggleBlueBox);
</script>

**Multiple scenarios:**

First we have several CSS animations that use `-webkit-animation-fill-mode: forwards;`.
And there is always a chance that an user decides to switch between tabs while something is animating.
The the user will experience the 100% CPU bug.

Second when logging in a user sees the sidebar animation with `-webkit-animation-fill-mode: forwards;` and later decides
to leave wercker open while doing some reading on a blog. Now if any CSS animation would start on wercker like a new feed message appears with a fadein.
While reading the user will experience the 100% CPU bug.

We have seen that in the first scenario the bug only appears when animating GPU rendered CSS styles.
In the second scenario is doesn't seem to mather, the only thing that mathers is that the first animation uses `-webkit-animation-fill-mode: forwards;`.

###Hope this post was usefull for you!
Luckily the fixes are on their way. In the current beta version of Chrome the little demo on this page is fixed.
And in the current Chrome Canary version the second scenario also seems to be fixed.
For now we're looking into using the CSS animation Events and using CSS3 Transitions.

## Earn some stickers of your own!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Thanks for tuning in, and there will be more posts next week! Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.


