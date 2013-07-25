---
title: "Improving golang build time with the wercker cache"
date: 2013-07-25
tags: golang, caching
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
published: false
---

<h4 class="subheader">
As we recently released the <a
href="http://blog.wercker.com/2013/07/22/Announcing-the-Open-Delivery-platform.html">Open
Delivery Platform</a> wercker is now capable of building and delivering
<a href="http://devcenter.wercker.com/articles/languages/go.html">Golang
projects</a>. In this post we want to take you through speeding up your
build time for golang projects with caching present on <a
href="http://wercker.com">wercker</a>.
</h4>

![image](http://f.cl.ly/items/0o2L381f1S3w2V1n2Q1o/EB41D256-9697-407A-9883-A1BE1AFF8674.jpg)

READMORE

Our previous post on Go dealt with [how to set up your golang projects
with wercker](http://blog.wercker.com/2013/07/10/Golang-on-wercker.html)
and [deploying them](http://blog.wercker.com/2013/07/10/deploying-golang-to-heroku.html) to Heroku
In this post I want to share how I improved the build time by using wercker's build cache.

## The build

Here are the steps including their duration [from one of the builds](https://app.wercker.com/#build/51dfef45bf67fc2f7500046a) 
of my own golang project called [httpcallback.io](https://github.com/pjvds/httpcallback.io):

1. get code (0 sec)
2. setup environment (15 sec)
3. environment variables (0 sec)
4. setup golang (0 sec)
5. **go get (55 sec)**
6. go build (0 sec)
7. go test (1 sec)
8. saving build output (0 sec)

Total time: 1 min 14 sec

This means that the `go get` step constitutes 75% of my build time.

## Wercker cache

Wercker has a per project build cache directory that is shared betweens builds.
A build can update the cache by writing to the `$WERCKER_CACHE_DIR` directory. If the build succeeds, this directory is saved when the build succeeds for future builds. We can levarage this to cache our Go workspace.

## Go workspace

The [golang box](https://github.com/pjvds/box-golang) that I have created to at Go support to wercker sets up and Go workspace in `$HOME/go`. This workspace contains the three required directories at its root:

* src contains Go source files organized into packages (one package per directory),
* pkg contains package objects, and
* bin contains executable commands.

When an build is finished, this workspace is filled with:

* The source code and compiled result of your project.
* The source code and cimpiled result of of the dependend packages.

The `go get` of future builds could be improved if this workspace is shared.

## Storing the workspace in cache

I added a script step to the end of my `wercker.yml` that rsync's the Go full workspace, but excludes the source directory.

  - script:
          name: Store cache
          code: |-
              rsync -avzv --exclude "$WERCKER_SOURCE_DIR" "$GOPATH/" "$WERCKER_CACHE_DIR/go-pkg-cache/"

## Populate cache

I added a script step that first checks if the cache directory is present, and if so it rsync's it into the Go workspace at the `$GOPATH`.

  - script:
      name: Populate cache
      code: |-
        # WARNING: If you do not use the pjvds/golang box:
        # before you copy and use this step in your own build pipeline
        # make sure you set $WERCKER_SOURCE_DIR to the package directory
        # or your project, like: $GOPATH/github.com/pjvds/httpcallback.io
          if test -d "$WERCKER_CACHE_DIR/go-pkg-cache"; then rsync -avzv --exclude "$WERCKER_SOURCE_DIR" "$WERCKER_CACHE_DIR/go-pkg-cache/" "$GOPATH/" ; fi

## The result

The `go get` now has a duration of 0 seconds, the actual time is a handful of millisecond, but bottom line we improved the build time with 55 seconds.

  1. get code (0 sec)
  2. setup environment (14 sec)
  3. environment variables (0 sec)
  4. setup golang (0 sec)
  5. **go get (0 sec)**
  6. go build (0 sec)
  7. go test (1 sec)
  8. saving build output (0 sec)

## What is next

Next is wrapping this logic in wercker steps, which I will release soon.
