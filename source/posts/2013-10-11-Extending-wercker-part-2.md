---
title: "Advanced: extending wercker - part 2"
date: 2013-10-11
tags: wercker, steps, heroku, wercker
author: Jacco Flenter
gravatarhash: 7d9ef3d3f6911e6e4f9c51f6d99c48f8
---


<h4 class="subheader">
    Wercker is an open platform and can be extended and enhanced by utilizing
    easily reusable steps and boxes.
</h4>

You can find them in the <a href="https://app.wercker.com/#explore">wercker directory</a>.
What if you can't find what you are looking for? Create your own. So let's see
how to test a step in the second part of this in-depth article.

READMORE

#### Introduction

In [part 1](/2013/10/11/Extending-wercker-part-1.html) we looked at a step that
communicated with an external service in order to set an environment variable
with a build number. This part will look into one way of testing the step
before deploying.

For our test, we could do test a number of ways:

* run the test against an external server. But since we want control over the
data on the server, this is not ideal.
* mock the api calls. This means we probably would need to test the python
script seperately. Although this would work, I wanted to avoid using too much
python. That way more people can tweak the step if they want to.
* setup a server locally and test against that instance.

I opted for the last solution. So our build will run on a python box and do the
following:

* validate the wercker-step.yml
* download the version_service project
* setup an instance of version_service and preload it with our test fixture.
* setup the environment variables, call run.sh and test the result.

To easily setup the environment variables, there's a tool called
[envdir](https://github.com/jezdez/envdir). It is a Python port of
[daemontools'](http://cr.yp.to/daemontools.html)
tool [envdir](http://cr.yp.to/daemontools/envdir.html).
Which is available on [pypi](https://pypi.python.org/pypi/envdir), so we can
simply use pip install to get it. What does it do?

> envdir runs another program with a modified environment according to files in
a specified directory.

#### Preparing the step

The directory structure from part 1 should look like:

``` text
|
+- wercker-step.yml
+- run.sh
+- fetch.py
```

We need to add some directories:

* `env/case1`. Here we will store the files for envdir
* `test`. Here we will store our fixture and test cases. We will also use it as
the location to run our tests from. This is similar to a step running
on wercker during a normal build/deploy. The current working directory during
a test is the source folder of the app and that is not where our fetch.py will
be located.

Now we need to create the right files so envdir can setup the environment
variables for us.

The environment we want to have is as follows:

``` sh
WERCKER_GENERATE_VERSION_API_KEY=e075a65470a66f40be1983d00a73876a9ba9dd45
WERCKER_GENERATE_VERISON_BASE_URL=http://localhost:8000
WERCKER_GENERATE_VERSION_FOR_APP=1
WERCKER_GENERATE_VERSION_IGNORE_BRANCHES=false
WERCKER_GENERATE_VERSION_username=admin
WERCKER_GIT_BRANCH=master
WERCKER_GIT_COMMIT=228b4257cbb797460563ba8806503f8512676df6
WERCKER_STEP_ROOT=..
```

This means that for each variable we need to create a file with the same name with
the text after the equal sign as the content (i.e. file
`env/case1/WERCKER_STEP_ROOT` contains nothing but two dots)

Next up is the fixture. You can easily create a fixture in django by running
the `python manage.py dumpdata` command. The details on how to do this are out
of scope for this article. We will use the fixture provided
[here](https://raw.github.com/flenter/step-generate-version/master/test/test_fixture.json).

Our test script `test/case1.sh` will call run.sh with a couple of different
variables and test for exit code and the environment variable `GENERATED_BUILD_NR`
like so:

``` sh

#!/usr/bin/env bash

echo "Case 1: server running. Script should not return an error."

echo ""
echo "Initial environment variables: "
export
echo ""
source $WERCKER_STEP_ROOT/run.sh

RESULT=$?

if [[ $RESULT != "0" ]] || [[ $GENERATED_BUILD_NR != "1" ]]; then
    echo "Test: FAIL"
    return 1 2>/dev/null || exit 1
else
    echo "Test: OK"
fi

echo ""
echo "! changing commit hash !"
TMP_WERCKER_GIT_COMMIT=$WERCKER_GIT_COMMIT
WERCKER_GIT_COMMIT+="_new_commit"

echo "new commit hash: $WERCKER_GIT_COMMIT"
echo ""

source $WERCKER_STEP_ROOT/run.sh

RESULT=$?

if [[ $RESULT != "0" ]] || [[ $GENERATED_BUILD_NR != "2" ]]; then
    echo "Test: FAIL"
    return 1 2>/dev/null || exit 1
else
    echo "Test: OK"
fi

echo ""
echo "! changing branch !"
TMP_WERCKER_GIT_BRANCH=$WERCKER_GIT_BRANCH
WERCKER_GIT_BRANCH="feature/this"

echo "new branch $WERCKER_GIT_BRANCH"
echo ""

source $WERCKER_STEP_ROOT/run.sh

RESULT=$?

if [[ $RESULT != "0" ]] || [[ $GENERATED_BUILD_NR != "1" ]]; then
    echo "Test: FAIL"
    return 1 2>/dev/null || exit 1
else
    echo "Test: OK"
fi

echo ""
echo "! Querying original branch/commit hash again. !"
WERCKER_GIT_COMMIT=$TMP_WERCKER_GIT_COMMIT
WERCKER_GIT_BRANCH=$TMP_WERCKER_GIT_BRANCH

source $WERCKER_STEP_ROOT/run.sh

RESULT=$?

if [[ $RESULT != "0" ]] || [[ $GENERATED_BUILD_NR != "1" ]]; then
    echo "Test: FAIL"
    return 1 2>/dev/null || exit 1
else
    echo "Test: OK"
fi

echo "Tests: complete"
```



Translating all this into a `wercker.yml` results in:

``` yaml
box: wercker/python
build:
  steps:
    - validate-wercker-step
    - script:
        name: install envdir
        code: sudo pip install envdir
    - script:
        name: clone and start server.
        code: |
          cd /tmp
          git clone https://github.com/flenter/versioning_service.git
          cd versioning_service
          python -m virtualenv --distribute venv
          source venv/bin/activate
          pip install -r requirements.txt
          cp $WERCKER_SOURCE_DIR/test/test_fixture.json initial_data.json
          ./manage.py syncdb --noinput
          ./manage.py run_gunicorn -D
    - script:
        name: scenario 1. service up
        code: |
          cd test
          envdir ../envs/case1 ./case1.sh
```

This is an example of just one way of testing a step with an external service.
We hope by showing you how we've implemented done this, you will further
enhance, write better versions of this step or completely different ones.

That's all for now!

---

Have fun!


### Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).

_note: The step [generate-version on github.com](https://github.com/flenter/step-generate-version) contains
more tests_
