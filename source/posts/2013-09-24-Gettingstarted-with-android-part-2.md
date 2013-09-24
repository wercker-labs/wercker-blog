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
displayed. We'll do this by adding an android emulator on our wercker
environment and run some instrument tests.

We need to update our project so we can access the TextView in our
mainActivity. For this we need to open `activity_main.xml` and add an
android:id attribute with the value: "@+id/introText" to the textview element
in `activity_main.xml`.

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

Now we can get the textview element in our code by calling `findViewById` on
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

        assertTrue("Check intro text", introText.equals("Hello world!"));
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
what happens on wercker and will explain the magic happening in the wercker.yml
later (step 3 continous with the wercker specific explanation).


Let
Go to tools -> Android -> SDK Manager. Check the checkbox for add Android 4.3
(API 18) with its underlying components and click install.

![Android's SDK manager](/images/posts/android-part2/sdk-install.jpg)

Next up: create and start the emulator instance.

Go to tools -> Android -> AVD Manager.
![Andoird's virtual device manager](/images/posts/android-part2/avd-manager.jpg)

Add a new virtual device by clicking new and create a new 'wercker' phone like so:

![new virtual device](/images/posts/android-part2/avd-new.jpg)

_note: you may be able to use hardware acceleration by using a different CPU on your machine see [the official android documentation](http://developer.android.com/tools/devices/emulator.html#acceleration)_

Select the wercker phone in the manager and start it.
![new virtual device](/images/posts/android-part2/avd-start.jpg)
The default settings will do just fine for our test. Launch the emulator.

![android emulator booting](/images/posts/android-part2/avd-booting.jpg)

Booting the emulator will take some time. When you finally see the lock screen,
fire up the terminal and go to the root of your application.

``` bash
$ gradle connectedInstrumentTest
Relying on packaging to define the extension of the main artifact has been deprecated and is scheduled to be removed in Gradle 2.0
:GettingTested:preBuild UP-TO-DATE
:GettingTested:preDebugBuild UP-TO-DATE
:GettingTested:preReleaseBuild UP-TO-DATE
:GettingTested:prepareComAndroidSupportAppcompatV71800Library UP-TO-DATE
:GettingTested:prepareDebugDependencies
:GettingTested:compileDebugAidl UP-TO-DATE
:GettingTested:compileDebugRenderscript UP-TO-DATE
:GettingTested:generateDebugBuildConfig UP-TO-DATE
:GettingTested:mergeDebugAssets UP-TO-DATE
:GettingTested:mergeDebugResources
:GettingTested:processDebugManifest
:GettingTested:processDebugResources
:GettingTested:generateDebugSources
:GettingTested:compileDebug
Note: [your location]/GettingTestedProject/GettingTested/src/main/java/com/wercker/gettingstarted/tests/MainActivityTest.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
:GettingTested:dexDebug
:GettingTested:processDebugJavaRes UP-TO-DATE
:GettingTested:validateDebugSigning
:GettingTested:packageDebug
:GettingTested:assembleDebug
:GettingTested:preTestBuild UP-TO-DATE
:GettingTested:prepareTestDependencies
:GettingTested:compileTestAidl UP-TO-DATE
:GettingTested:compileTestRenderscript UP-TO-DATE
:GettingTested:processTestTestManifest
:GettingTested:generateTestBuildConfig UP-TO-DATE
:GettingTested:mergeTestAssets UP-TO-DATE
:GettingTested:mergeTestResources UP-TO-DATE
:GettingTested:processTestResources
:GettingTested:generateTestSources
:GettingTested:compileTest
:GettingTested:dexTest UP-TO-DATE
:GettingTested:processTestJavaRes UP-TO-DATE
:GettingTested:packageTest
:GettingTested:assembleTest
:GettingTested:connectedInstrumentTest
```


---

Have fun!

### Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).
