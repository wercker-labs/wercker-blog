---
title: Build and deploy status notifications with CCtray.xml
date: 2013-07-10
tags: opendelivery, cctray, notifications
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
published: false
---

# Build and deploy status notifications with CCtray.xml

Today we have added build and deployment status output in the cctray.xml format, an RRS-like CI server build status xml feed, which is originally developed for CruiseControl.net. In this post I would like to share the details and how I leverage this to get notified on important status changes via CCMenu.

## Configuration
Wercker offers two endpoints per project that serve status information in the cctray.xml format. One for build status and one for deployment status.

### Buids status feed

The feed url that contains the build status from an project is:

	https://app.wercker.com/api/v2/applications/{PROJECT-ID}/cc/build

You need to replace `{PROJECT-ID}` with the project id of the project you want to monitor.

### Deploy status feed

The feed url that contains the deployment status of an project's deploy target is:

	https://app.wercker.com/api/v2/applications/{PROJECT-ID}/cc/deploytargets/{DEPLOY-TARGET-NAME}

You need to replace `{PROJECT-ID}` with the project id of the project you want to monitor and `{DEPLOY-TARGET-NAME}` with the name of the deploy target.

## Tools

Here is a list of tools that allow you to monitor cctray.xml feeds and that can be used to monitor your build and deployment statusses of your wercker projects.

* [CCMenu for Mac](http://ccmenu.sourceforge.net/)
* [CCTray for Windows](http://confluence.public.thoughtworks.org/display/CCNET/CCTray)
* [buildnotify for Linux](https://bitbucket.org/Anay/buildnotify/wiki/Home)
* [CruiseControl Monitor for Firefox](https://addons.mozilla.org/en-US/firefox/addon/cruisecontrol-monitor/)

## Using CCMenu

There are a lot of tools that support the cc status format. But, since I'm working on a Mac most of the time I decided to use [CCMenu](http://ccmenu.sourceforge.net/). It allows you to monitor multiple projects at the same time and identifies the overall build status with a glance to the menu bar.

[!image](/images/ccmenu/tray.png)

Download and install the latest version of ccmenu from the [CCMenu project](http://sourceforge.net/projects/ccmenu/files/CCMenu/).

### Finding your project id

Navigate to your project page at [app.wercker.com](https://app.wercker.com) and copy your project id from the url.

![image](/images/ccmenu/project_id.png)

### Add builds to CCMenu

Open CCMenu by clicking on the icon in the top menu bar and open preferences.

![image](/images/ccmenu/open_preferences.png)

### Enter feed details

Click on the plus sign in the preference window and enter your project cc feed url. It is important to follow the following steps closely because CCMenu was acting a bit weird when I entered my details.

1. Enter your feed url
2. Click `Use URL as entered above` option
3. Enter your wercker credentials
4. Check `The server requires authentication`
5. Click `Continue`
6. See the status of your project in your menu bar!

![image](/images/ccmenu/add_feed.png)

### Other feeds

You can repeat the previous steps for other feeds as well.
