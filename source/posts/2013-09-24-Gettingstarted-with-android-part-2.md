---
title: Getting started with Android - part 2
date: 2013-09-24
tags: android
author: Jacco Flenter
gravatarhash: 7d9ef3d3f6911e6e4f9c51f6d99c48f8
published: true
---


<h4 class="subheader">
In <a href="https://blog.wercker.com/2013/09/19/Gettingstarted-with-android-part-1.html">part I</a> of this series we dove into how to get started with a simple <a href="http://www.android.com/">Android</a> application using android studio version 0.2.9. For part 2 it's time to add some tests</h4>

![wercker and android](/images/posts/android/wanda.jpg)

You can sign up for wercker for free
[here](https://app.wercker.com/users/new/).

READMORE


### Prerequisites

* A [GitHub](https://github.com/) or [Bitbucket](http://bitbucket.org) account
with a pre-made repository that will hold your code.
* You have
[registered for a wercker account](https://app.wercker.com/users/new) and
[signed in](https://app.wercker.com/users).
* [Android studio](http://developer.android.com/sdk/installing/studio.html) is
already installed on your machine.
* The project created in [part 1](http://blog.wercker.com/2013/09/19/Gettingstarted-with-android-part-1.html) already added to wercker.
* Basic Java and android development knowledge.

### Recap of part 1 ###
In part 1 we created a getting started project, using nearly all the default
options of android studio's new project wizard. The project consists of one
main activity and displays the text "Hello world!" on screen.

### Step 1: Adding tests

Let's add to our project a test to see if "Hello World!" really is
displayed. We'll do this by adding an android emulator to our wercker
environment and run some tests using android's instrumentation framework on an
emulated android environment.

To make this possible, we need to update our project so we can access the
TextView in our mainActivity. We need to modify the `activity_main.xml` and add
an android:id attribute with the value: "@+id/introText" to the textview
element in `activity_main.xml`.

``` xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin"
    tools:context=".MainActivity">

    <!-- modified element: android:id="@+id/introText" added -->
    <TextView
        android:id="@+id/introText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/hello_world" />

</RelativeLayout>
```

We can get the textview element in our code by calling `findViewById` on
our main activity.

Now that we can access our textview, it's time to create the test. Add a new
package named 'tests' to com.wercker.gettingstarted and create a new class
named 'MainActivityClass'

``` java
package com.wercker.gettingstarted.tests;

import android.test.ActivityInstrumentationTestCase2;
import android.widget.TextView;

import com.wercker.gettingstarted.MainActivity;
import com.wercker.gettingstarted.R;

/**
 * Created by jacco @ wercker on 9/23/13.
 */
public class MainActivityTest extends ActivityInstrumentationTestCase2<MainActivity> {

    public MainActivityTest() {
        super("com.wercker.gettingstarted", MainActivity.class);
    }


    public void testWelcomeText() {
        MainActivity activity;
        activity = (MainActivity) getActivity();

        TextView tView;
        tView = (TextView) activity.findViewById(R.id.introText);
        assertNotNull(tView);

        String introText;
        introText = tView.getText().toString();
        assertNotNull(introText);

        assertTrue("Check intro text", introText.equals("Hello universe!"));
    }
}
```

Our test class is compact and - apart from the constructor which helps creating
the MainActivity instance for our test - contains only one interesting method:
`testWelcomeText`. In this method we retrieve the activity, get the text view
and finally verify the introText.

### Step 2: Preparing the emulator

We're going to add the emulator to our sdk, create an 'android virtual device'
(emulated android device) and start it. This step is not required, but shows
what happens on wercker and will explain the magic happening in the [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/)
later (step 3 continues with the wercker specific explanation).


Lets
Go to tools -> Android -> SDK Manager. Check the checkbox for add Android 4.3
(API 18) with its underlying components and click install.

![Android's SDK manager](/images/posts/android-part2/sdk-install.jpg)

Next up: create and start the emulator instance.

Go to tools -> Android -> AVD Manager.
![Android's virtual device manager](/images/posts/android-part2/avd-manager.jpg)

Add a new virtual device by clicking new and create a new 'wercker' phone like this:

![new virtual device](/images/posts/android-part2/avd-new.jpg)

_note: you may be able to use hardware acceleration by using a different CPU on your machine see [the official android documentation](http://developer.android.com/tools/devices/emulator.html#acceleration)_

Select the wercker phone in the manager and start it.
![new virtual device](/images/posts/android-part2/avd-start.jpg)
The default settings will do just fine for our test. Launch the emulator.

![android emulator booting](/images/posts/android-part2/avd-booting.jpg)

Booting the emulator will take some time. When you finally see the lock screen,
fire up the terminal and go to the root of your application and run our
instrument test.

``` text
$ gradle connectedInstrumentTest
Relying on packaging to define the extension of the main artifact has been deprecated and is scheduled to be removed in Gradle 2.0
:gettingstarted:preBuild UP-TO-DATE
:gettingstarted:preDebugBuild UP-TO-DATE
:gettingstarted:preReleaseBuild UP-TO-DATE
:gettingstarted:prepareComAndroidSupportAppcompatV71800Library UP-TO-DATE
:gettingstarted:prepareDebugDependencies
:gettingstarted:compileDebugAidl UP-TO-DATE
:gettingstarted:compileDebugRenderscript UP-TO-DATE
:gettingstarted:generateDebugBuildConfig UP-TO-DATE
:gettingstarted:mergeDebugAssets UP-TO-DATE
:gettingstarted:mergeDebugResources UP-TO-DATE
:gettingstarted:processDebugManifest UP-TO-DATE
:gettingstarted:processDebugResources UP-TO-DATE
:gettingstarted:generateDebugSources UP-TO-DATE
:gettingstarted:compileDebug
Note: [your location]/gettingstartedProject/gettingstarted/src/main/java/com/wercker/gettingstarted/tests/MainActivityTest.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
:gettingstarted:dexDebug
:gettingstarted:processDebugJavaRes UP-TO-DATE
:gettingstarted:validateDebugSigning
:gettingstarted:packageDebug
:gettingstarted:assembleDebug
:gettingstarted:preTestBuild UP-TO-DATE
:gettingstarted:prepareTestDependencies
:gettingstarted:compileTestAidl UP-TO-DATE
:gettingstarted:compileTestRenderscript UP-TO-DATE
:gettingstarted:processTestTestManifest UP-TO-DATE
:gettingstarted:generateTestBuildConfig UP-TO-DATE
:gettingstarted:mergeTestAssets UP-TO-DATE
:gettingstarted:mergeTestResources UP-TO-DATE
:gettingstarted:processTestResources UP-TO-DATE
:gettingstarted:generateTestSources UP-TO-DATE
:gettingstarted:compileTest
:gettingstarted:dexTest UP-TO-DATE
:gettingstarted:processTestJavaRes UP-TO-DATE
:gettingstarted:packageTest UP-TO-DATE
:gettingstarted:assembleTest UP-TO-DATE
:gettingstarted:connectedInstrumentTest

com.wercker.gettingstarted.tests.MainActivityTest > testWelcomeText[wercker(AVD) - 4.3] FAILED
    junit.framework.AssertionFailedError: Check intro text
    at com.wercker.gettingstarted.tests.MainActivityTest.testWelcomeText(MainActivityTest.java:32)
:gettingstarted:connectedInstrumentTest FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':gettingstarted:connectedInstrumentTest'.
> There were failing tests. See the report at: file:///[your location]/gettingstartedProject/gettingstarted/build/reports/instrumentTests/connected/index.html

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

Total time: 13.105 secs
```

So now that we've tested it, we can see the provided test was broken.
We can easily fix it by chainging the code in the MainActivityTest class from
`introText.equals("Hello world!")` to `introText.equals("Hello universe!")`.

If we run it again now:

``` text
$ gradle connectedInstrumentTest
Relying on packaging to define the extension of the main artifact has been deprecated and is scheduled to be removed in Gradle 2.0
:gettingstarted:preBuild UP-TO-DATE
:gettingstarted:preDebugBuild UP-TO-DATE
:gettingstarted:preReleaseBuild UP-TO-DATE
:gettingstarted:prepareComAndroidSupportAppcompatV71800Library UP-TO-DATE
:gettingstarted:prepareDebugDependencies
:gettingstarted:compileDebugAidl UP-TO-DATE
:gettingstarted:compileDebugRenderscript UP-TO-DATE
:gettingstarted:generateDebugBuildConfig UP-TO-DATE
:gettingstarted:mergeDebugAssets UP-TO-DATE
:gettingstarted:mergeDebugResources
:gettingstarted:processDebugManifest
:gettingstarted:processDebugResources
:gettingstarted:generateDebugSources
:gettingstarted:compileDebug
Note: [your location]/gettingstartedProject/gettingstarted/src/main/java/com/wercker/gettingstarted/tests/MainActivityTest.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
:gettingstarted:dexDebug
:gettingstarted:processDebugJavaRes UP-TO-DATE
:gettingstarted:validateDebugSigning
:gettingstarted:packageDebug
:gettingstarted:assembleDebug
:gettingstarted:preTestBuild UP-TO-DATE
:gettingstarted:prepareTestDependencies
:gettingstarted:compileTestAidl UP-TO-DATE
:gettingstarted:compileTestRenderscript UP-TO-DATE
:gettingstarted:processTestTestManifest
:gettingstarted:generateTestBuildConfig UP-TO-DATE
:gettingstarted:mergeTestAssets UP-TO-DATE
:gettingstarted:mergeTestResources UP-TO-DATE
:gettingstarted:processTestResources
:gettingstarted:generateTestSources
:gettingstarted:compileTest
:gettingstarted:dexTest UP-TO-DATE
:gettingstarted:processTestJavaRes UP-TO-DATE
:gettingstarted:packageTest
:gettingstarted:assembleTest
:gettingstarted:connectedInstrumentTest
BUILD SUCCESSFUL

Total time: 12.467 secs
```
Now our build is successful.
_note: You can also see the test being run on the screen of the emulated android device._

### Step 3: Update the wercker.yml

In [part 1](https://blog.wercker.com/2013/09/19/Gettingstarted-with-android-part-1.html) we created
the following wercker.yml:

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
            echo $ANDROID_BUILD_TOOLS
            echo $ANDROID_UPDATE_FILTER
      # A step that executes gradle build command
      - script:
          name: run gradle
          code: |
            gradle --full-stacktrace -q --project-cache-dir=$WERCKER_CACHE_DIR build```
```

We need to expand it so it will:

1. install the emulator
2. create an AVD
3. fires up the AVD
4. runs the connectedInstrumentTest task


Adding the emulator to the sdk is easy, we can do this by adding the
[android-sdk-update step](https://app.wercker.com/#applications/52406677f92d800d4b001f3d/tab/details):

```
    - android-sdk-update:
        filter: sysimg-18
```

The code above will add the system image for android 4.3 to our sdk.
Next up are numbers two and three on our list. For this there's also a step
available called [setup-android-emulator](https://app.wercker.com/#applications/5241920d4f6b6b786f000586/tab/details).

```
    - setup-android-emulator:
        target: android-18
```

Will create an AVD with the name wercker and launch it, wait for it to be ready.
The final thing we need to do is run the gradle task connectedInstrumentTest (or cIT for short).

```
    - script:
        name: run connected instrument test
        code: |
            gradle cIT -q
```

In summary the wercker.yml will be:

```
box: wercker/android
# Build definition
build:
  # The steps that will be executed on build
  steps:
    - android-sdk-update:
        filter: sysimg-18
    - setup-android-emulator:
        target: android-18
    - script:
        name: run gradle checkConnected
        code: |
          gradle --project-cache-dir=$WERCKER_CACHE_DIR cIT
    - script:
        name: run gradle build
        code: |
          gradle --full-stacktrace --project-cache-dir=$WERCKER_CACHE_DIR build
```

Next, add it to the repository and watch the magic happen on wercker.

```
$ git add .
$ git commit -am "instrument test added"
$ git push
```

Depending on whether your MainActivityTest class wants the textview to say
"Hello world!" or "Hello universe", you should get either a green or a red
build.

In part 3 of this post we will go into a different way of testing android apps
on wercker. Stay tuned.

---

Have fun!

### Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).
