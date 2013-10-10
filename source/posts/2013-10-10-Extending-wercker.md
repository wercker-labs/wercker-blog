---
title: Extending wercker with an external service
date: 2013-10-10
tags: wercker steps heroku wercker
author: Jacco Flenter
gravatarhash: 7d9ef3d3f6911e6e4f9c51f6d99c48f8
---


<h4 class="subheader">
    wercker is an open platform and can be extended and enhanced by utilizing
    easily reusable steps and boxes.
</h4>

You can find them in the <a href="https://app.wercker.com/#explore">wercker directory</a>.
What if you can't find what you are looking for? Create your own. So let's see
how to create a step in this in-depth article.

READMORE

#### Introduction

While creating [the](/2013/09/19/Gettingstarted-with-android-part-1.html)
[series](/2013/09/24/Gettingstarted-with-android-part-2.html)
[on](/2013/09/27/Gettingstarted-with-android-part-3.html)
[android](/2013/10/04/Getting-started-with-android-part-4.html) development I
found that I wanted wercker to set the version numbering for me. Each release
should out increment some counter. In this in-depth article, we will explore
a solution using an external website (running on a free heroku instance) which
can supply us with a build number. A working version of the step on wercker can
be found
[here](https://app.wercker.com/#applications/524d763ba5db0adc70010666/tab/details)


#### The basics

What is a step? We envision steps as basic blocks of functionality which are
contained and might be reused for different applications. So instead of using
the script step for everything, you can create a step for actions that are
a bit more complex or that may use certain options/switches you otherwise have
to copy/paste from that other project. Typically steps do one of the following:

* notify users (for instance via
[email](https://app.wercker.com/#applications/520c938f8a20a2624501003e/tab/details),
[irc](https://app.wercker.com/#applications/5235dd91dfe78bb24c001b1f/tab/details),
[campfire](https://app.wercker.com/#applications/51f2a3e8df5a46247c000e0d/tab/details)
[flowdock](https://app.wercker.com/#applications/5232a884341102d86800611b/tab/details)).
* install software. Example:
the [bundle-install](https://app.wercker.com/#applications/51c829d13179be44780020be/tab/details)
step
* run tests, validate code or run some kind of lint. Like the
[validate-wercker-step](https://app.wercker.com/#applications/51cd593c7578aa5b5300026b/tab/details)
* generate an environment variable or store some data in a certain location.
The [add-ssh-key](https://app.wercker.com/#applications/523afff01aa016c8590015b1/tab/details)
step does this

Our step will create an environment variable that we can use later during the
build and update whichever file we need to for our specific language.

### The pipeline

Whenever you start a build or a deploy there are a couple of basic steps that
are set in stone in the wercker pipeline:

* get code. During builds this is a git clone, during deploy the build output
is retrieved.
* setup environment. During this step the boxes are started. Based on the
wercker.yml
* environment variables. This step defines the default wercker environment
variables as well as any defined in the deploy target or in settings / pipeline.
* saving build output (builds only)

In our wercker.yml, there are two opportunities in the pipeline to insert our
own logic, which are defined in the wercker yaml as:

* `steps`. This is run right after `environment variables`
* `after-steps`. They are run after a build/deploy is finished (succesfully or
not).

The steps are executed in order and if one fails, the next ones after are not
executed. The after-steps are different, they are basically run outside of the
build flow and are meant for steps that need to be run, no matter the outcome
of the build or deploy (or specifically when a build/deploy is failed).
This is particularly useful for sending notifications.

### Helpers and definitions

As defined in the introduction of this article, we want to increment some
counter and optionaly want to do this per branch. Wercker provides a lot of
information by default that we can use in our steps. So let's look at a couple
of them.

<table border="0">
<tr>
    <th>VARIABLE NAME</th>
    <th>EXAMPLE VALUE</th>
    <th>PURPOSE</th>
</tr>
<tr>
    <td>WERCKER_GIT_OWNER</td>
    <td>wercker</td>
    <td>The owner of the repository</td>
</tr>
<tr>
    <td>WERCKER_GIT_REPOSITORY</td>
    <td>step-bundle-install</td>
    <td>The name of the repository</td>
</tr>
<tr>
    <td>WERCKER_GIT_BRANCH</td>
    <td>master</td>
    <td>The branch name</td>
</tr>
<tr>
    <td>WERCKER_GIT_COMMIT</td>
    <td>ef306b2479a7ecd433
        7875b4d954a4c8fc18
        e237</td>
    <td>The commit hash</td>
</tr>
<tr>
    <td>WERCKER_SOURCE_DIR</td>
    <td>$WERCKER_ROOT/src</td>
    <td>The path to the directory of the source code</td>
</tr>
<tr>
    <td>WERCKER_CACHE_DIR</td>
    <td>/cache</td>
    <td>The path to the cache directory. This directory will be stored after the pipeline completes and restored when the pipeline runs again</td>
</tr>
<tr>
    <td>WERCKER_STEP_ROOT</td>
    <td>/wercker/steps/wercker
        /bundle-install/0.9.1</td>
    <td>The path to the working directory of the step that is currently executed. It contains the full content as deployed to the <a href="https://app.wercker.com/#explore">wercker directory</a></td>
</tr>
<tr>
    <td>WERCKER_STEP_ID</td>
    <td>9c182f44-e12d-4daf-91eb-a48d0540cc10</td>
    <td>The unique idenfier for the step, unique for each build/deploy.</td>
</tr>
<tr>
    <td>WERCKER_STEP_NAME</td>
    <td>bundle-install</td>
    <td>The name of the step as specified by the step in <strong>wercker-step.yml</strong></td>
</tr>
<tr>
    <td>WERCKER_REPORT_MESSAGE_FILE</td>
    <td>$WERCKER_REPORT_DIR/
        $WERCKER_STEP_ID/
        message.txt</td>
    <td>The location of a file you can use to output additonal information about the step.</td>
</tr>
</table>

We have all of the variables we need to determine our build number: the
branch, the commit hash and the application name. Do we need an external server
for getting this build number? Can we not create a database or simple text file
and store it in the wercker cache? The answer to that, yes. It would however
not behave exactly as we want. The biggest problem is, that the cache of a build
is only stored on a succesful build. Treating the cache as persistant storage
that we can access each build might also not be the best idea. Fortunately
there's a simple django application called
[versioning_service](https://github.com/flenter/versioning_service) which can
help us get the correct build numbers.

### Communicating with the service

The django application provides a RESTful interface which allows a user to
query for a
build number based on the application, branch name and commit hash. An example
version is running on: [buildnr.herokuapp.com](http://buildnr.herokuapp.com)

Back to our rest call, what does a user of the step need to do? The django app
requires a user to register, so random people can't increment your build
numbers artificially. It also requires a user to create an application.

In detail we need to know a number of things in order to get our build number:

* app id.
* the branch name
* commit hash
* user credentials: the django application allows for a username and api_key to
be specified

### The definition of a step

Defining a step is done in the `wercker-step.yml` file. There we can specify
the name, version and parameters of the step:

``` yaml
name: generate-version
version: 0.0.1-alpha
description: Generate build number
keywords:
  - version
  - build number
properties:
  api_key:
    type: string
    required: true
  username:
    type: string
    required: true
  for_app:
    type: number
  ignore_branches:
    type: boolean
    default: false
  base_url:
    type: string
    default: http://buildnr.herokuapp.com
```

Since commit hash and branch name are already environment variables defined
earlier in the pipeline, the steps' user doesn't need to specify them. There
are however two optional properties defined which make the step a bit more
flexible: ignore_branches and base_url. Ignore branches basically will set the
branch name to a fixed string, so the build count goes up no matter which
branch the user is working on. Base_url is to allow a user to specify his/her
own instance.

The properties specified in our wercker-step.yml will be available in our step
as `$WERCKER_[step name]_[property name]`. In our case this means our step can
expect `$WERCKER_GENERATE_VERSION_API_KEY` to be a string. If  can see a
couple of conversions happened:

* dashes are converted to underscores.
* all characters are transformed to uppercase.

All right, now that we know the name of the step and our properties we can
start writing logic. The core of a step is `run.sh`. At wercker we prefer to
implement as much of our steps in bash. Since not every machine may have the
language of Our `run.sh` however will call a small python script `fetch.py` to
retrieve the json and return a number. Let's look at the first part of the
python script, which contains the handling of the command options:

``` python
#!/usr/bin/env python

from optparse import OptionParser
import urllib2
import sys


parser = OptionParser()

parser.add_option("-U", "--url", type="string", dest="url")
parser.add_option("-u", "--user", type="string", dest="username")
parser.add_option("-k", "--key", type="string", dest="key")
parser.add_option("-a", "--appId", type="string", dest="app")
parser.add_option("-b", "--branch", type="string", dest="branch")
parser.add_option("-c", "--commit", type="string", dest="commit")

(options, args) = parser.parse_args()

if options.url.endswith("/") is False:
    options.url += "/"

url = "{url}api/v1/branch_version?format=json&".format(url=options.url)

url += "username={user}&api_key={key}".format(
    user=options.username,
    key=options.key)

url += "&for_app={app}&for_branch={branch}&commit_hash={commit}".format(
    app=options.app,
    branch=options.branch,
    commit=options.commit
)
```

The second part retrieves the content and returns either a string or a non 0
exit for the bash script to handle.

``` python
try:
    f = urllib2.urlopen(url)

    status_code = f.getcode()

except urllib2.HTTPError as e:
    status_code = e.code
except urllib2.URLError as e:
    sys.exit("""Error: Failed to reach server.
Reason: {reason}""".format(reason=e.reason))

if status_code == -1:
    sys.exit("Unable to connect to {url}".format(url=url))
elif status_code != 200:
    sys.exit(
        """Server couldn't fulfill the request.
url: {url}
status code: {code}""".format(
        url=url,
        code=status_code
    ))
else:
    import json

    data = json.load(f)
    meta = data.get("meta")
    if meta and meta.get("total_count") == 1:
        obj = data.get("objects")[0]
        version_number = obj.get("version_number")
        print version_number
    else:
        sys.exit("Unexpected return data: " + data)
```

A step is by default executed in either the $WERCKER_SOURCE_DIR. And this means
that for our run.sh to call fetch.py we need to refer to it's full path (or
change directory). There is a $WERCKER_STEP_ROOT environment variable. So our
`run.sh` will look like:

``` bash
#!/usr/bin/env bash

if [ "$WERCKER_GENERATE_VERSION_IGNORE_BRANCHES" = "false" ]
then
    GENERATED_BUILD_NR=$($WERCKER_STEP_ROOT/fetch.py -U $WERCKER_GENERATE_VERSION_BASE_URL -u $WERCKER_GENERATE_VERSION_USERNAME -k $WERCKER_GENERATE_VERSION_API_KEY -a $WERCKER_GENERATE_VERSION_FOR_APP -b $WERCKER_GIT_BRANCH -c $WERCKER_GIT_COMMIT)
else
    GENERATED_BUILD_NR=$($WERCKER_STEP_ROOT/fetch.py -U $WERCKER_GENERATE_VERSION_BASE_URL -u $WERCKER_GENERATE_VERSION_USERNAME -k $WERCKER_GENERATE_VERSION_API_KEY -a $WERCKER_GENERATE_VERSION_FOR_APP -b master -c $WERCKER_GIT_COMMIT)
fi

if [[ "$GENERATED_BUILD_NR" = "" ]]; then
    echo ""
    echo "Failed to get build number" 1>&2

    return 1 2>/dev/null || exit 1
else
    echo "\$GENERATED_BUILD_NR: $GENERATED_BUILD_NR"
fi
```

This is the minimum amount of code for our wercker generate-version step. To get
them in the wercker directory, you need to:

1. add the files to a repository and please add a README.md.
2. create a *public* application on wercker.
3. Push your code.
4. Add a 'Wercker directory' deploy target
5. Go to the build and click deploy (to wercker directory).

After this you can use your step by adding following code to an apps' steps:

``` yaml
        - your_user_name/generate-version:
            api_key: your_api_key
            username: your_username
            for_app: app_id
```

Have fun!

---


### Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).
