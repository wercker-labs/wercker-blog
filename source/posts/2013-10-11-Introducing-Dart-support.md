---
title: Introducing Dart support
date: 2013-10-11
tags: dart, dartlang
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
---
<h4 class="subheader">
I am very pleased to announce Dart support at wercker! In this post I quickly want to share the details of our environment. You can expect some tutorial posts shortly.
</h4>

![Overheating MacBook pro](/images/posts/dart/wercker+dart.jpg)

READMORE

## Dart environment

Support for Dart is released in the form of a [wercker box](http://devcenter.wercker.com/articles/boxes/) called [wercker/dart](https://app.wercker.com/#applications/5255489a367392913001326b/tab/details). This box has has everything in place to build and release your Dart projects. It inherits from the [wercker/ubuntu12.04-webessentials](https://app.wercker.com/#applications/51ab0c42df8960ba45003fd9/tab/details) box, that means that it has all the packages, tools and dependencies available for build and testing web applications.

The box runs Dart version 0.8.1.2_r28355 which installed in `$HOME/DART_SDK`. This path is available via `DART_SDK` and included in the `PATH`:

```
export DART_SDK="$HOME/dart-sdk"
export PATH="$PATH:$DART_SDK/bin"
```

Dart provides the following tools:

* [`dart`](https://www.dartlang.org/docs/dart-up-and-running/contents/ch04-tools-dart-vm.html): The standalone VM
* [`dart2js`](https://www.dartlang.org/docs/dart-up-and-running/contents/ch04-tools-dart2js.html): The Dart-to-JavaScript compiler
* [`dartanalyzer`](https://www.dartlang.org/docs/dart-up-and-running/contents/ch04-tools-dart_analyzer.html): The static analyzer
* [`dartdoc`](https://www.dartlang.org/docs/dart-up-and-running/contents/ch04-tools-dartdoc.html): The API documentation generator
* [`pub`](http://pub.dartlang.org/): The Dart package manager

## Install dependencies

The Dart SDK ships with package manager [Pub](http://www.dartlang.org/docs/pub-package-manager/). Execute `pub install` to download all dependencies defined in the `pubspec.yaml` file.

## Example application

To demo the Dart box I've forked the Minesweeper clone in Dart ["Pop, Pop, Win!"](https://github.com/pjvds/pop-pop-win) and added a [wercker.yml](https://github.com/pjvds/pop-pop-win/blob/master/wercker.yml) to define the build pipeline.

``` yaml
box: wercker/dart
build:
  steps:
    - script:
        name: build
        code: |
          pub install
    - script:
        name: test
        code: |
          dart --checked tool/hop_runner.dart test
```

This pipeline consists of two script steps that are executed chronologically. The first step installs all dependencies with the `pub install` command. The seconds step executes the tests by executing the `hop_runner.dart` file.

## Resources

Here are the resources related to this sample project:

* [Sample project source code at GitHub](https://github.com/pjvds/pop-pop-win/)
* [Sample project at wercker](https://app.wercker.com/project/bykey/a4bb9e6ebb162598e26ce5aff19243e3)
* [Dart box at wercker directory](https://app.wercker.com/#applications/5255489a367392913001326b/tab/details)

## Earn some stickers of your own!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Thanks for tuning in, and there will be more posts next week! Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.
