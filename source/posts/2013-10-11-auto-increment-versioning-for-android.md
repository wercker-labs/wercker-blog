---
title: Autoincrement versioning for android
date: 2013-10-11
tags: android
author: Jacco Flenter
gravatarhash: 7d9ef3d3f6911e6e4f9c51f6d99c48f8
---


<h4 class="subheader">
What if you want each release of your build to automatically set a new version
code and name?
</h4>

READMORE

There's a
[generate-version step](https://app.wercker.com/#applications/524d763ba5db0adc70010666/tab/details)
on wercker, which can help us with versioning. It uses a
[small app](https://app.wercker.com/#applications/524ae392a5db0adc700012d2/tab)
deployed on a free instance of heroku:
[buildnr.herokuapp.com](http://buildnr.herokuapp.com) which you can use (or of
course deploy your own).

_note: the app and step are not official wercker services, but meant as example
tools. That's why they are hosted external from wercker and completely open
source and free._

### What does the step do?

According to the documentation:

> Sets an environment variable to contain a build nr. This step depends on a
running instance of (buildnr.herokuapp.com)[http://buildnr.herokuapp.com] or your own instance of the python/django app: (github.com/flenter/versioning_service)[https://github.com/flenter/versioning_service]

> For this step you need to:

> login/register on (buildnr.herokuapp.com)[http://buildnr.herokuapp.com]
add an application (which is not much more than a name)
go to the details of the application and you will find the values for the parameters of this step.

The output of the step is an environment variable named `$GENERATED_BUILD_NR`.

What do we need to do to make this useful for us? In order use this environment
variable we will modify the build.gradle file so we can set both the
versionCode and versionName. After that we add the step to the `wercker.yaml`
and add the parameters to the gradle build command.

### Updating our build.gradle file

Let's start with updating the build.gradle file. The changes will allow us to
specify the versionCode and versionName attributes in the AndroidManifest.xml.

Add this simple script to your build.gradle file, add it above the `android`
section.

``` text
/**
 * Get the version code from command line param
 *
 * @return int If the param -PversionCode is present then return int value or -1
 */
def setVersionCode = { ->

    def code = project.hasProperty('versionCode') ? versionCode.toInteger() : -1
    println "VersionCode is set to $code"
    return code
}

def setVersionName = { ->

    def code = project.hasProperty('versionName') ? versionName : "0.0.0"
    println "VersionName is set to $code"
    return code
}
```

### Updating the wercker.yml

There's a example code in the readme of the step:

``` yaml
build:
  steps:
    - flenter/generate-version:
        api_key: e5da976d9f679e38c2faa04d3ecc92f3485e1517
        username: admin
        for_app: 1
```

However, it is nicer to use environment variables instead of username and api
key in our `wercker.yml`. This specifically applies to open source projects.

You can use environment variables in your wercker.yml by just writing
`$ENVIRONEMT_VARIALBE`. We'll call our environment variable `VERSIONING_API_KEY`.
So combining the step with the additional parameters for the build, your wercker file will end up looking like this:

``` yaml
box: wercker/android
# Build definition
build:
  # The steps that will be executed on build
  steps:
    - flenter/generate-version:
        api_key: $VERSIONING_API_KEY
        username: admin
        for_app: 2
    - script:
        name: run gradle robolectric
        code: |
          gradle robolectric -i --project-cache-dir=$WERCKER_CACHE_DIR
    - script:
        name: run gradle build
        code: |
          gradle build --full-stacktrace --project-cache-dir=$WERCKER_CACHE_DIR build -PversionCode=$GENERATED_BUILD_NR -PversionName=1.0.$GENERATED_BUILD_NR
```

### Create an account

To use this step you need to register and add an app on
[buildnr.herokuapp.com](http://buildnr.herokuapp.com).

![Login page](/images/posts/android-versioning/login.jpg)

On the details page of the app you can find the parameters for the step. We can
now update the `wercker.yaml` file with your user information. Keep the
the value `VERSIONING_API_KEY` for property api_key.

We can set this value in the settings of your application on wercker: Go to
pipeline and click `+ add new variable`. Create a protected environment
variable with the name `VERSIONING_API_KEY` and use the api_key value as can be
found on your apps' details page on buildnr.herokuapp.com.

![add environment variable](/images/posts/android-versioning/environment_variable.jpg)


### Finish

Commit all all changes and enjoy the result!

If you are interested in how the step works: the construction and workings are
also described in two articles on this blog:

* [Advanced: extending wercker - part 1](/2013/10/11/Extending-wercker-part-1.html)
* [Advanced: extending wercker - part 2](/2013/10/11/Extending-wercker-part-2.html)

---

Have fun!

### Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).
