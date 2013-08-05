---
title: Checking test coverage with gocov
date: 2013-08-05
tags: gocov, golang
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
published: false
---

Code coverage can be an important metric to watch. It gives you insight in which parts of your code are covered well by tests and which can have some extra attention. In this post I will explain how I leverages gocov to add test coverage reports to the build pipeline of one of my go projects.

## Executing tests with gocov

To measure test coverage in Go we can use [gocov](https://github.com/axw/gocov) created by [Andrew Wilkins](http://awilkins.id.au/). It has a `test` command that executes the test with the default `go test` test runner and generate the coverage details in json format. The `gocov test` command respects the `go test` commands and still prints output from it to the console. Here is an example of a script step that runs the tests for all packages and sub package of the working directory and writes the coverage results to `coverage.json`:

    - script:
        name: Test
        code: |-
          # Get gocov package
            go get github.com/axw/gocov/gocov

            # Execute actual tests and store coverage result
            gocov test ./... > coverage.json

Next to the `coverage.json` that will be create, it writes the following output:

  ok    github.com/pjvds/go-cqrs/sourcing 0.070s
  ok    github.com/pjvds/go-cqrs/storage  0.023s
  ok    github.com/pjvds/go-cqrs/storage/eventstore 0.022s
  ok    github.com/pjvds/go-cqrs/storage/memory 0.012s
  ?     github.com/pjvds/go-cqrs/storage/serialization  [no test files]
  ok    github.com/pjvds/go-cqrs/tests  0.021s
  ?     github.com/pjvds/go-cqrs/tests/domain [no test files]
  ?     github.com/pjvds/go-cqrs/tests/events [no test files]

You can see the actual step result on wercker: [go-cqrs / 6c8cd61 / test](https://app.wercker.com/#buildstep/51ffb8a9170dc79a480004e1).

## Generating the report

The `coverage.json` created by the previous step can now be used as input for the `report` command. This commands generates a textual report based on the `coverage.json`. It makes sence to do this in a seperate step to seperate the test output and the coverage output. Here is an example of a script step that executes the `gocov report` command:

  - script:
        name: Coverage
        code: gocov report coverage.json

This step will write the following output:

  github.com/pjvds/go-cqrs/storage/eventstore/Log.go     InitLogging     100.00% (3/3)
  github.com/pjvds/go-cqrs/storage/eventstore/EventStore.go  EventStore.ReadStream   0.00% (0/27)
  github.com/pjvds/go-cqrs/storage/eventstore/EventStore.go  EventStore.WriteStream  0.00% (0/22)
  github.com/pjvds/go-cqrs/storage/eventstore/EventStore.go  downloadEvent     0.00% (0/13)
  github.com/pjvds/go-cqrs/storage/eventstore/EventStore.go  processFeed     0.00% (0/9)
  github.com/pjvds/go-cqrs/storage/eventstore/EventStore.go  linksToMap    0.00% (0/4)
  github.com/pjvds/go-cqrs/storage/eventstore/EventStore.go  DailEventStore    0.00% (0/1)
  github.com/pjvds/go-cqrs/storage/eventstore      ----------------------  3.80% (3/79)

  github.com/pjvds/go-cqrs/tests/Log.go  InitLogging   100.00% (3/3)
  github.com/pjvds/go-cqrs/tests     -----------   100.00% (3/3)

You can see the actual step result on wercker: [go-cqrs / 6c8cd61 / coverage](https://app.wercker.com/#buildstep/51ffb8a9170dc79a480004e2).

## Writing to the artifact dir

The `coverage.json` can also be used to create a nice self sufficient html report. We can use [gocov-html](https://github.com/matm/gocov-html) tool for this. Here is how I enchanced the previous step and added html reporting that is stored in the artifact directory.

    - script:
        name: Coverage
        code: |-
            go get github.com/matm/gocov-html
            gocov report coverage.json
            gocov-html coverage.json > $WERCKER_REPORT_ARTIFACTS_DIR/coverage.html

When this step succeeds you can download the artifacts package and open `coverage.html`.

![coverage report](http://f.cl.ly/items/3L160B140h222X3w3s1C/Screen%20Shot%202013-08-05%20at%205.23.27%20PM.png)

## What is next

Now wercker gives insight in your test coverage for your go projects. The next step is to use the coverage as a number to pass or fail the build. Stay tuned for an update where we will create a step to do this.
