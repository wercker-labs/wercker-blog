---
title: Running Jasmine tests headless
date: 2013-09-27
tags: deployment, golang
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
---

[Jasmine](http://pivotal.github.io/jasmine/) is a populair [behavior-driven development](http://dannorth.net/introducing-bdd/) framework for testing JavaScript code. In this post I want to share how to run these test in a headless fashion on wercker.

READMORE

## Jasmine

[Jasmine](http://pivotal.github.io/jasmine/) is a populair [behavior-driven development](http://dannorth.net/introducing-bdd/) framework for testing JavaScript code. It is developed and maintained by the great people at [pivotal labs](http://pivotallabs.com/). Here is an example of a Jasmine test:

``` javascript
describe("A suite", function() {
  it("contains spec with an expectation", function() {
    expect(true).toBe(true);
  });
});
```

Jasmine is built in JavaScript and must be included into a JS environment, such as a web page, in order to run.
Jasmine itself comes with a great static html page which runs all your tests.
There are also [many small packages](https://github.com/pivotal/jasmine/wiki) for specific environments to dynamically generate this runner.

## Jasmine Headless Webkit

Running Jasmine when you need to test code that will run in a browser environment can be slow.
It still needs to render every pixel in a browser. This is why [John Bintz](http://johnbintz.com/) created
the [`jasmine-headless-webkit` gem](http://johnbintz.github.io/jasmine-headless-webkit/).
Jasmine-headless-webkit uses the [QtWebKit widget](http://trac.webkit.org/wiki/QtWebKit) to run
your specs without ever needing to render a pixel. It's nearly as fast as running in a JavaScript engine.

## Example project

To show how to execute Jasmine tests in a headless browser on wercker I needed a project that contains a good amount
of Jasmine tests.
I remember [some great posts](http://tinnedfruit.com/2011/04/26/testing-backbone-apps-with-jasmine-sinon-3.html)
from [Jim Newbery](http://tinnedfruit.com/) and [forked](https://github.com/pjvds/backbone-jasmine-examples) his
sample project.

It is a Ruby on Rails application, but the steps I describe should be independent from your application stack.

## Jasmin tests

The sample project contains a folder [`spec/`](https://github.com/pjvds/backbone-jasmine-examples/tree/master/spec)
that holds the specs. This is a default convention for jasmine. The `jasmine.yml` can be found in the default location [`spec/javascripts/support/jasmine.yml`](https://github.com/pjvds/backbone-jasmine-examples/blob/master/spec/javascripts/support/jasmine.yml) and defines properties like where the spec and source files are located.

## Adding the wercker.yml

After I cloned the project I added a [`wercker.yml`](http://devcenter.wercker.com/articles/werckeryml/):

``` yaml
box: wercker/blank
# Build definition
build:
  # The steps that will be executed on build
  steps:

    # Install required packages for jasmine-headless-webkit gem
    # If these packages are already available on the machine the installation
    # will be skipped.
    - install-packages:
        packages: libqt4-dev qt4-qmake

    # We assume that you have setup jasmine in your source repository
    # and will use jasmine-headless-webkit for the execution of the tests.
    - script:
        name: Install jasmine-headless-webkit
        code: |
          sudo gem install jasmine-headless-webkit --no-ri --no-rdoc

    - script:
        name: Test
        code: |
          sudo xvfb-run jasmine-headless-webkit -c
```

The first line defines that the build will be execute in the `wercker/default` box.
This is a virtual machine that isn't specific to any stack or software. It does however have Ruby, Node and Python available, so installing and executing gems doesn't require you to install Ruby first.

You should pick the right box for your project.
You can browse the [wercker directory](https://app.wercker.com/#explore/boxes) or
[create your own](http://devcenter.wercker.com/articles/boxes/bash.html).

In the build pipeline the first step that is defined is the `install-packages` step.
This will install the packages `libqt4-dev` and `qt4-qmake`, which are required by the `jasmine-headless-webkit` gem.
If these packages are already available, for example when you use the `wercker/ruby` box, the installation is skipped.

After we made sure the correct packages are available the `jasmine-headless-webkit` gem is installed.
I used the `--no-ri` and `--no-rdoc` option to speed up the installation process.

Finally, there is a script step that will run the `jasmine-headless-webkit` command.
It uses the `xvfb-run` command to execute it inside a context where there is a virtual frame buffer.
This is like attaching a monitor to the machine, which is required to execute the tests.
I specify the `-c` option to get a nice collored output.

## The result

Here is a screenshot of my green build at wercker:

[![](/images/posts/running-jasmine-tests-headless/green_build.png)](https://app.wercker.com/#build/5245968ff5f6947e71000228)

## Links

* [Jasmine homepage](http://pivotal.github.io/jasmine/)
* [Jasmine-headless-webkit gem](http://johnbintz.github.io/jasmine-headless-webkit/)
* [Sample project at github](https://github.com/pjvds/backbone-jasmine-examples)
* [The project at wercker](https://app.wercker.com/#applications/524573f3c4b717064b007473)

### Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/).
