---
title: Getting started with Android - part 1
date: 2013-09-19
tags: android
author: Jacco Flenter
gravatarhash: 7d9ef3d3f6911e6e4f9c51f6d99c48f8
published: true
---


<h4 class="subheader">
In part I of this post we'll dive into how to get started with a simple <a href="http://www.android.com/">Android</a> application using android studio version 0.2.9.</h4>

![wercker and android](/images/posts/android/wanda.jpg)

You can sign up for wercker for free
[here](https://app.wercker.com/users/new/).

READMORE


### Prerequisites

* A [GitHub](https://github.com/) or [Bitbucket](http://bitbucket.org) account with a pre-made repository that will hold your code.
* You have [registered for a wercker account](https://app.wercker.com/users/new) and [signed in](https://app.wercker.com/users).
* [Android studio](http://developer.android.com/sdk/installing/studio.html) is already installed on your machine.
* Basic Java and android development knowledge.

### Creating a new project

Go to the file menu and select: new project. You will be prompted with a wizard:

![step 1](/images/posts/android/step1.png)

Step 1: For this tutorial we've opted for GettingStatred and updated the package name to start with com.wercker.

Remember where you store the project, we need it for some git commands later.

![step 2](/images/posts/android/step2.png)

For the next few steps you can basically, click next. The default options will suffice.

![step 3](/images/posts/android/step3.png)
![step 4](/images/posts/android/step4.png)

In android studio **you may get a error** about an unresolvable dependency. Android studio will disable the dependency in the build.gradle file and so we will ignore it.

![possible error message](/images/posts/android/step5.png)

### Git

Before adding our project to get let's create (if it doesn't exist) or modify the
`.gitignore` in the root of the GettingStartedProject:

``` text
# built application files
*.apk
*.ap_

# files for the dex VM
*.dex

# Java class files
*.class

# generated files
bin/
gen/

# Local configuration file (sdk path, etc)
local.properties
.idea/workspace.xml
.gradle
.DS_Store
```

Now we can safely initialize the repository as documented on github/bitbucket. After that, let's add it to wercker.

### Add project to wercker

Add your GitHub or Bitbucket project to wercker using the wercker [add application flow](https://app.wercker.com/#project/create).
When you add the application to wercker, you should see the following default `wercker.yml` for android applications:

``` yaml
box: wercker/android
# Build definition
build:
  # The steps that will be executed on build
  steps:
    - script:
        name: show base information
        code: |
          gradle -v
          echo $ANDROID_HOME
          echo $ANDROID_SDK_VERSION
          echo $ANDROID_BUILD_TOOLS
          echo $ANDROID_UPDATE_FILTER
    # A step that executes `gradle build` command
    - script:
        name: run gradle
        code: |
          gradle --full-stacktrace -q --project-cache-dir=$WERCKER_CACHE_DIR build
```

You don't have to add the yaml to your application, but let's add it. Later we will expand it.

```
$ git add wercker.yml
$ git commit -am "wercker yaml added"
$ git push
```

Switching back to your application on wercker, you will see a new build. Open the build and view the information each individual step gives. The "show base information" step shows information about the tools installed on the android box:

* which sdk items are installed.
* which versions of the build tools are available.
* which version of gradle is installed.

The "run gradle" step returns very little, since it is run in quiet mode.

After a while your build should be finished and green. However there's little to be seen of the build result. So let's see if we can explore that a little bit.

Edit your wercker.yml file and add the following to the end of the file:

```
  after-steps:
    # Use the build results
    - script:
        name: inspect build result
        code: |
          ls -la GettingStarted/build/apk/
          cp GettingStarted/build/apk/*.apk $WERCKER_REPORT_ARTIFACTS_DIR
```

Now we have a step that will show the files in the application (in my case named GettingStarted) build/apk folder and store them as build artifacts. This all happens after the build is completed and don't count for the success/failure of a build. Let's see it in action, by pushing our code.

```
$ git commit -am "after step added"
$ git push
```

In the "inspect build result" step details we should see (amongst some other information):

```
$ls -la GettingStarted/build/apk/
total 52
drwxrwxr-x  2 ubuntu ubuntu  4096 Sep 19 10:34 .
drwxrwxr-x 12 ubuntu ubuntu  4096 Sep 19 10:34 ..
-rw-rw-r--  1 ubuntu ubuntu 21843 Sep 19 10:34 GettingStarted-debug-unaligned.apk
-rw-rw-r--  1 ubuntu ubuntu 19461 Sep 19 10:34 GettingStarted-release-unsigned.apk
```

And the artifacts tab should contain a link to an archive with the apks.


That's it for now. In <a href="http://blog.wercker.com/2013/09/24/Gettingstarted-with-android-part-2.html">part 2</a> of this post we will go into testing of android apps on wercker.

---

Have fun!

### Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).
