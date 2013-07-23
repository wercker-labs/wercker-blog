---
title: Spotlight on wercker pipeline steps
date: 2013-07-23
tags: opendelivery, boxes, steps
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
---

<h4 class="subheader">

Yesterday we <a href="http://blog.wercker.com/2013/07/22/Announcing-the-Open-Delivery-platform.html">announced<a/> the wercker <a href="http://app.wercker.com/#explore">directory</a> and the Open Delivery platform, enabling every developer to create their own box on wercker that specifies their stack but also the <a href="http://devcenter.wercker.com/articles/steps/">steps that are integral to their <a href="http://devcenter.wercker.com/articles/introduction/pipeline.html">pipeline</a> on wercker.

In this post we want to do a deepdive into steps and how they work on wercker, thus paving the way for you to create your own!
</h4>

This article is also available at our [dev center](http://devcenter.wercker.com/articles/steps/guide.html).

READMORE

## Wercker steps

Steps make up the wercker pipeline and can either be executed in the build or
deploy phase within the pipeline. Examples of a build step include compilation of
your code, running your unit tests or performing jshint. A deploy step could be
the synchronization of static assets, for which we've created the s3sync step,
that takes some Amazon Web Services credentials and bucket information and
places these assets on Amazon S3.

![image](http://f.cl.ly/items/2O3V2n3A1n2d3u3S363D/wercker_pipeline.png)

## wercker-step.yml

Every step must contain a `wercker-step.yml` file, which is the manifest that
describes the properties for the step.

Here is an example of a `wercker-step.yml` that only holds the required fields:

``` yaml
name: create-file
version: 0.9.6
description: create-file step
```

You can also add keywords to your step which increases discoverability:

``` yaml
keywords:
  - file
  - text
  - create
```

## Step entry point

Every step is executed by executing a `run.sh` file, which should be present as well.
This file is responsible for the high-level
organization of the step's functionality. The actual step logic can be written
inside the `run.sh`. When you want to group things you can move your logic
to multiple shell scripts and call them from `run.sh`. You could also develop
a step in Ruby, Python or Node.js and use the `run.sh` to bootstrap this. A good example
of the latter is the [validate-wercker-step](https://github.com/wercker/step-validate-wercker-step).

## Step options

Steps can have options or parameters to receive input. For example, the `create-file` step
has the option `filename` that specifies the filename and where the file should be created.
Options are set as elements of the step attribute in `wercker.yml`. Here is an
example that uses the `create-file` step and specifies three options:

    - create-file:
        name: generate production robots.txt
        filename: ./_production/robots.txt
        content: |-
          User-agent: *
          Allow: /

The `name` option is default for every step and it allows the user to specify the
logical name for that step. In the example above `filename` and `content` are
`create-file` specific options. The value from options can be retrieved with the
`get_option` function:

    filename=`get_option filename`
    echo "Value for filename option: $filename"

## Environment variables

The following environment variables are available in the context of a step execution:

Note: environment variables that contain a path to a directory contain the resolved path and ends with the directory name without a slash.


<table border="0">
<tr>
    <th>VARIABLE NAME</th>
    <th>EXAMPLE</th>
    <th>PURPOSE</th>
</tr>
<tr>
    <td>WERCKER_ROOT</td>
    <td>/pipeline/build</td>
    <td>The root folder where wercker runs the build or deployment pipeline</td>
</tr>
<tr>
    <td>WERCKER_SOURCE_DIR</td>
    <td>$WERCKER_ROOT/src</td>
    <td>The path to the directory of the source code</td>
</tr>
<tr>
    <td>WERCKER_OUTPUT_DIR</td>
    <td>/output</td>
    <td>The path to the directory that contains, or will contain, the output of the build pipeline</td>
</tr>
<tr>
    <td>WERCKER_CACHE_DIR</td>
    <td>/cache</td>
    <td>The path to the cache directory. This directory will be stored after the pipeline completes and restored when the pipeline runs again
</tr>
<tr>
    <td>WERCKER_STEP_ROOT</td>
    <td>/wercker/steps/wercker/bundle-install/0.9.0</td>
    <td>The path to the working directory of the step that is currently executed. It contains the full content as deployed to the <a href="http://app.wercker.com/#explore">wercker directory</a>
</tr>
<tr>
    <td>WERCKER_STEP_ID</td>
    <td>S3SYNC7</td>
    <td>The unique - within the context of the pipeline execution - idenfier for the step. The pattern is _{STEPNAME}{ORDINAL}. The value could be different on the next run of the pipeline
</tr>
<tr>
    <td>WERCKER_STEP_NAME</td>
    <td>S3SYNC</td>
    <td>The name of the step as specified by the step in <strong>wercker-step.yml</strong>
</tr>
</table>

## Writing output

The following functions are available for writing output:

`success` - writes a success message.

`fail` - writes a failure message and **stops execution**.

`warn` - writes a warning message.

`info` - writes a informational message.

`debug` - writes a debug message.

Here is a short example:

    debug "checking if config exists…"
    if [ -e ".config" ]
    then
        info ".config file found"
    else
        fail "missing .config file"
    fi

## Using wercker cache

A cache directory is shared between builds. The path to this directory is stored in the environment variable `$WERCKER_CACHE_DIR`. A step can leverage this cache to share assets between builds. A good example of this is the [`bundler-install`](https://app.wercker.com/#applications/51c829d13179be44780020be/tab/details) step which leverages the cache to shorten the installation time of dependencies. It does so by storing the end-result of downloading and compiling dependend packages. Future builds can use this as a starting point and only new dependencies which were not cached are downloaded. At the start of every build the cache directory is filled with the cached content from the last successful build, if not older than 24 hours.

A step should not depend on content of the cache and should be able to handle scenarios where the cache is not populated.

Here is a simple example of a step that leverages the cache:

``` bash
if [ -f "$WERKER_CACHE_DIR/mystep/a-dependency.bin" ];
then
    debug "a-dependency.bin found in cache"
else
    debug "tool.rar not found in cache, will download"
    curl -o "$WERCKER_CACHE_DIR/mystep/a-dependency.bin" "http://domain.org/a-dependency.bin"
fi
```

This example checks for the file `$WERKER_CACHE_DIR/mystep/a-dependency.bin` from the cache. If the file is not found, it will be downloaded to the cache directory so it will be available for future builds, if the build succeeds.


## Check if a variable is set and not empty

    if [ ! -n "$var" ] ; then
        echo "var is not set or value is empty"
    fi

Where `$var` is the variable you want to check.

## Check if command exists

    if ! type s3cmd &> /dev/null
    then
        echo "s3cmd exists!"
    else
        echo "s3cmd does not exist!"
    fi

Where `s3cmd` is the command you want to check. The `&> /dev/null` part makes it silence (it generates no output).

## Concluding

We hope this post has helped you in gaining a better understanding of steps on wercker. We created a simple guide that creates a **Campfire Notification step** for your builds that you can view [here](http://devcenter.wercker.com/articles/steps/create.html).

For inspiration you can view our official [GitHub page](http://github.com/wercker) that contains a lot of steps currently present on wercker.
And of course don't forget to explore the [wercker directory](http://app.wercker.com/#explore) as well!

![image](http://f.cl.ly/items/3A153L0J1Z1i0t081V0N/Screen%20Shot%202013-07-23%20at%2010.00.35%20AM.png)

## Earn some stickers!

Let us know about the steps you create for wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.
