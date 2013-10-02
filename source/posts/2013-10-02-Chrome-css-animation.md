---
title: CSS animations in Chrome
date: 2013-10-02
tags: chrome css animation
author: Lindsey Bateman
gravatarhash: e1c82876f21cdafafd2b01a1e625f587
published: true
---
<h4 class="subheader">
Fixing 100% cpu utilization with CSS animations in Chrome
</h4>

We've had animations in wercker for a while, but recently we implemented
several new ones. We added fades for freshly added items on the feed and the
sidebar animated in when loading the page.
Fairly simple CSS animations to make wercker come to life and give it richer
feel.

All fun until a user gave (much appreciated) feedback, that ...
READMORE

![Overheating MacBook pro](/images/posts/chromecssanimations/heatmap.jpg)

Having wercker open in your browser could result in 100% CPU utilization on
Chrome. Small disclaimer: we have not investigated which version of chrome
first had this problem. However, we see the bug with both versions 29.0.1547.76
and 30.0.1599.66 (released today, 2nd of October).

Back to the our bug: at first we couldn't reproduce it for several reasons:

* our local testing and staging environments wouldn't have new items
appearing all the time.
* usually you test with the tab open and active.

That last one was especially annoying since it turned out it was when wercker
was not running in the foreground. Focussing on the wercker tab would cause the
CPU load to drop to a normal level.

![CSS animation experiment in Chrome](/images/posts/chromecssanimations/chrometaskmanager.png)

After some research it seemed that a couple of the CSS animations were to blame.

And although there are several scenarios which can cause problems, one of the
most prominent animations on our site was the one for the sidebar
menu. It slides in after signing in, in order to enhance the feeling that the
sidebar is something linked to you as a user (it contains your profile picture
and several links to your profile and app lists).
For this animation we use **animation keyframes** and the
`-webkit-animation-fill-mode: forwards;` so that the sidebar retains its
position from the last frame of the animation. An animation normally would snap
back to its original setting after it is finished playing.

This is great for us, but not for Chrome. The CSS animations using
`-webkit-animation-fill-mode: forwards;` are causing the 100% CPU load bug.
It actually only occurs when the CSS animation ends and the tab
is inactive.

## Demo

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

<div class="demo">
	<div class="red__block"></div>
</div>

Let's toggle an rotating animation on the red semi-circle and switch to an
other or a new tab before the 3 second animation is finished playing.

<a class="js-toggle-play button">Toggle animation</a>

<script type="text/javascript">
	var toggleBlueBox = function(){
		$('.red__block').toggleClass("active");
	};
	$(".js-toggle-play").on("click", toggleBlueBox);
</script>

### Deconstructing the demo

There actually are several scenarios where we can trigger the problem, but the
above is closest to what happened on wercker.

The html looks like:

``` html
	<div class="demo">
		<div class="red__block"></div>
	</div>
```

With the following styles:

``` css
	.demo {
		margin: 40px;
	}

	.red__block {
		width: 50px;
		height: 50px;
		border: 5px solid red;
		border-color: red red transparent transparent;
		border-radius: 25px;
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
```

And on the toggle button we have a simple toggleClass action.

``` javascript
	var toggleBlueBox = function(){
		$('.red__block').toggleClass("active");
	};
	$(".js-toggle-play").on("click", toggleBlueBox);
```

In this demo we used rotate, on our website we used translate-3d. These are
effects running on the GPU in chrome. Using something like border-width (CPU
based animations) in our tests would not result in the high cpu load.

### The second scenario:

In the first scenario the bug only appears when animating GPU accelerated CSS
animations. In the second scenario it doesn't matter if it's GPU or CPU based.
See the [second demo](http://jsbin.com/egeDUCo/7/edit?html,css,js,output) in action
on jsbin.

This time we need two animations, so our HTML looks like this:

``` html
<div class="demo">
	<div class="blue__block"></div>
	<div class="red__block"></div>
</div>
```

And the following css:

``` css
.blue__block {
  width: 120px;
  height: 120px;
  background-color: blue;
  border: 1px solid yellow;
  padding: 99px;
  -webkit-animation: grow 2s linear;
  -webkit-animation-fill-mode: forwards;
}

.red__block {
  width: 100px;
  height: 100px;
  padding: 99px;
  background-color: red;
  border: 1px solid green;
  -webkit-animation: grow 2s linear;
}

@-webkit-keyframes grow {
     from{
        border-width: 1px;
        padding: 99px;
     }
     to{
        border-width: 99px;
		padding: 1px
    }
}
```

### Implications

We need to be carefull using animations especially in conbimation with
`-webkit-animation-fill-mode: forwards`. Most of the animations we use on the
website can also be achieved using transitions which - as far as we've seen -
don't suffer from the same issues.

And also: test whether your animations cause 100% cpu load when they are
finished playing and don't be surprised if the cpu load spikes when the tab is
not active!

Luckily the fixes for chrome are on their way:
* the current beta version of chrome doesn't have any trouble with the demo on
this page
* the Chrome Canary release has no problems with both demo's.

### Hope this post was usefull to you!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Thanks for tuning in, and there will be more posts next week! Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.


