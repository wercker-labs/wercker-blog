---
title: Getting started with Android - part 3
date: 2013-09-27
tags: android
author: Jacco Flenter
gravatarhash: 7d9ef3d3f6911e6e4f9c51f6d99c48f8
published: true
---


<h4 class="subheader">
After exploring <a href="http://blog.wercker.com/2013/09/24/Gettingstarted-with-android-part-2.html">running tests with the android emulator</a>, in this part, we will explore a different road for testing android apps.</h4>

![wercker and android](/images/posts/android-part3/wanda_robo.jpg)

This is part 3 in the series "getting started with Android", so please see also:

* <a href="/2013/09/19/Gettingstarted-with-android-part-1.html">part 1</a> how to build a simple Android application using android studio version 0.2.9.
* <a href="/2013/09/24/Gettingstarted-with-android-part-2.html">part 2</a> how to test that simple application

READMORE

### Introduction ###
In the [second part](/2013/09/24/Gettingstarted-with-android-part-2.html) of this
series we added a test to our "getting started project". Unfortunately setting
it all up and waiting for the emulator to fully start each build takes up quite
some time. Here's where [robolectric](http://pivotal.github.io/robolectric/)
can help us out.

From their website:
> Robolectric is a unit test framework that de-fangs the Android SDK so you can
> test-drive the development of your Android app.

It allows us to test android apps without using the Android emulator,
saving valuable time.

### Prerequisites

This guide expands on code from
[part 2](/2013/09/24/Gettingstarted-with-android-part-2.html) (which expands on
part 1). So please go through both parts before continuing.a

### Step 1: Configuring your tools

If we want to use robolectric, unfortunately we need to change configurations
for both the editor and gradle.

Starting with gradle, let's modify our current `gradle.build` file in the
[project root]/GettingStarted directory.

``` text
buildscript {
    repositories {
        mavenCentral()

        // Added: maven section. Adds a location for the robolectric plugin
        maven {
            url "https://oss.sonatype.org/content/repositories/snapshots"
        }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:0.5.+'
        // Added: robolectric to the classpath
        classpath 'com.stanfy.android:gradle-plugin-java-robolectric:2.0.2-SNAPSHOT'
    }
}

apply plugin: 'android'
// Added: the robolectric plguin
apply plugin: 'robolectric'

repositories {
    mavenCentral()
}

android {
    compileSdkVersion 18
    buildToolsVersion "17.0.0"

    defaultConfig {
        minSdkVersion 7
        targetSdkVersion 16
    }
}

dependencies {

  // You must install or update the Support Repository through the SDK manager to use this dependency.
  // The Support Repository (separate from the corresponding library) can be found in the Extras category.
  // compile 'com.android.support:appcompat-v7:18.0.0'

  // Added: robolectricCompile dependencies
  robolectricCompile group: 'org.robolectric', name: 'robolectric', version: '2.+'
  robolectricCompile group: 'junit', name: 'junit', version: '4.+'
}
```

_Note: the android support dependency may or may not be commented out. This depends on wether you have the android-support library installed._

You can check to see if all is working by running from the projects' root in
the terminal:

```
$ gradle tasks
:tasks

z------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Android tasks
-------------
androidDependencies - Displays the Android dependencies of the project
signingReport - Displays the signing info for each variant
...


Verification tasks
------------------
check - Runs all checks.
connectedCheck - Runs all device checks on currently connected devices.
connectedInstrumentTest - Installs and runs the tests for Build 'Debug' on connected devices.
deviceCheck - Runs all device checks using Device Providers and Test Servers.
lint - Runs lint on all variants.
robolectric - Runs the unit tests using robolectric.

To see all tasks and more detail, run with --all.

BUILD SUCCESSFUL

Total time: 5.763 secs
```

The output above was shortened a bit, but note that in the **verification
tasks** list there's now a new task available:
`robolectric`.

Now to configure Android studio, as it does not understand the
robolectricCompile directives in the gradle file, we need to add junit and
robolectric by hand.

Let's prepare our project. Create a `libs` directory in the root of the
project. Now fire up the browser and go to
[oss.sonatype.org/](https://oss.sonatype.org/).
Use the search to get to robolectric and click on 'jar' in the download column.
Search for junit and click on the 'jar' in the download column.

Copy both jars file to the `libs` directory.

Right/control click on 'GettingStarted' (not the parent project) and
`Open Module settings`. For both junit and robolectric:

1. click on the small + button
2. select `Jars or directory`
3. select one of the jars in the `libs` directory and click ok
4. change the scope of that dependency from Compile to Test.

![Add dependency](/images/posts/android-part3/add-jar.jpg)

### Step 2: adding tests

In the GettingStarted directory of your project, create the following
directory: `test` and inside that a `java` directory. Right click (or control
click for Mac users) on this `java` directory and mark it as test sources root.

In the java directory create a new package: `com.wercker.gettingstarted` and add a
class `MainActivityRoboTest` to this package. We'll edit it so it looks like:

``` java
package com.wercker.gettingstarted;

import android.widget.TextView;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.robolectric.Robolectric;
import org.robolectric.RobolectricTestRunner;

import static junit.framework.Assert.assertNotNull;
import static junit.framework.Assert.assertTrue;


/**
 * Created by jacco @ wercker on 9/26/13.
 */

@RunWith(RobolectricTestRunner.class)
public class MainActivityRoboTest {


    @Test
    public void shouldWelcome() throws Exception {

        MainActivity activity;
        activity = Robolectric.buildActivity(MainActivity.class).create().get();
        assertNotNull(activity);

        TextView tView;
        tView = (TextView) activity.findViewById(R.id.introText);
        assertNotNull(tView);

        String introText;
        introText = tView.getText().toString();
        assertNotNull(introText);

        assertTrue("Check intro text:", introText.equals("Hello universe!"));

    }
}
```

We're testing for the string "Hello universe!" again, so our test should fail.
Back to the terminal again:

``` text
$ gradle robolectric
Relying on packaging to define the extension of the main artifact has been deprecated and is scheduled to be removed in Gradle 2.0
:GettingStarted:preBuild UP-TO-DATE
:GettingStarted:preDebugBuild UP-TO-DATE
:GettingStarted:preReleaseBuild UP-TO-DATE
:GettingStarted:prepareComAndroidSupportAppcompatV71800Library
:GettingStarted:prepareDebugDependencies
:GettingStarted:compileDebugAidl
:GettingStarted:compileDebugRenderscript
:GettingStarted:generateDebugBuildConfig
:GettingStarted:mergeDebugAssets
:GettingStarted:mergeDebugResources
:GettingStarted:processDebugManifest
:GettingStarted:processDebugResources
:GettingStarted:generateDebugSources
:GettingStarted:compileDebug
Note: [your location]GettingStartedProject/GettingStarted/src/main/java/com/wercker/gettingstarted/tests/MainActivityTest.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.:GettingStarted:dexDebug
:GettingStarted:processDebugJavaRes UP-TO-DATE
:GettingStarted:validateDebugSigning
:GettingStarted:packageDebug
:GettingStarted:assembleDebug
:GettingStarted:prepareReleaseDependencies
:GettingStarted:compileReleaseAidl
:GettingStarted:compileReleaseRenderscript
:GettingStarted:generateReleaseBuildConfig
:GettingStarted:mergeReleaseAssets
:GettingStarted:mergeReleaseResources
:GettingStarted:processReleaseManifest
:GettingStarted:processReleaseResources
:GettingStarted:generateReleaseSources
:GettingStarted:compileRelease
:GettingStarted:dexRelease
:GettingStarted:processReleaseJavaRes UP-TO-DATE
:GettingStarted:packageRelease
:GettingStarted:assembleRelease
:GettingStarted:assemble
:GettingStarted:compileRobolectricJava
:GettingStarted:processRobolectricResources UP-TO-DATE
:GettingStarted:robolectricClasses
:gettingstarted:robolectric

com.wercker.gettingstarted.MainActivityTest > shouldWelcome FAILED
    java.lang.AssertionError at MainActivityTest.java:31

1 test completed, 1 failed
:GettingStarted:robolectric FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':GettingStarted:robolectric'.
> There were failing tests. See the report at: file:///[your location]/GettingStartedProject/GettingStarted/build/reports/tests/index.html

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

Total time: 14.815 secs
```

To use this on wercker we only have to add the following code snippet:

``` text
    - script:
        name: run gradle robolectric
        code: |
          gradle robolectric --project-cache-dir=$WERCKER_CACHE_DIR -i
```

We can also disable the emulator for our build, resulting in:

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
    - script:
        name: run gradle robolectric
        code: |
          gradle robolectric -i --project-cache-dir=$WERCKER_CACHE_DIR
    - script:
        name: run gradle build
        code: |
          gradle --full-stacktrace --project-cache-dir=$WERCKER_CACHE_DIR build
  after-steps:
    # Use the build results
    - script:
        name: inspect build result
        code: |
          ls -la GettingStarted/build/apk/
          cp GettingStarted/build/apk/*.apk $WERCKER_REPORT_ARTIFACTS_DIR
```

Time to push our changes to wercker:

```
$ git add .
$ git commit -am "Projet with failing robolectric test/step"
$ git push
```

On wercker, the build should fail on the 'run gradle robolectric' step.
Scrolling down through the log, we see nearly at the bottom:

```
> Building > :GettingStarted:robolectric> Building > :GettingStarted:robolectric> Building > :GettingStarted:robolectric> Building > :GettingStarted:robolectric> Building > :GettingStarted:robolectric.wercker.gettingstarted.MainActivityTest.shouldWelcome(MainActivityTest.java:31)
```

Similar to what we saw in part 2, the result of gradle on wercker is not ideal,
but it is something we are working on to improve.

Ok, let's fix the build by changing our MainActivityRoboTest to check for `Hello world!` instead of `Hello universe!` and enjoy how much faster our builds are.

That's all for now. Part 4 of this series will be about deploying. So stay tuned.

---

Have fun!

### Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).
