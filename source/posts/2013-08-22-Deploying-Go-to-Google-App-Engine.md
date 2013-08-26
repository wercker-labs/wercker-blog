---
title: Deploying Go to Google App Engine with wercker
date: 2013-08-22
tags: deployment, golang, AppEngine
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
---

<h4 class="subheader">
<a href="http://golang.org">Go</a> is one of my favorite languages. I like how it brings the many advantages from dynamic languages to the static world. Also Google App Engine is a mature platform that offers benefits such as switching versions, feature switches and a pack of various services out of the box. In this post I describe how I leverage wercker to deploy my Go application to App Engine.
</h4>

![image](http://f.cl.ly/items/0C1E2R1t0v3n101L1g25/77AFC437-D853-4973-A15B-6E9BBAEA6079.jpg)

READMORE

## The application

I've created a small [application](https://github.com/pjvds/go-cities-appengine/) in [Go](http://golang.org) that returns a list of cities as JSON on request. The application logic can be found in [`app.go`](https://github.com/pjvds/go-cities-appengine/blob/master/app.go):

``` go
package cities

import (
    "encoding/json"
    "net/http"
)

var (
    // The cities that we will serve
    cities = []string{
        "Amsterdam", "San Francisco", "Paris", "New York", "Portland",
    }
)

func init() {
    // Register the index handler to the
    // default DefaultServeMux.
    http.HandleFunc("/", handleIndex)
}

func handleIndex(rw http.ResponseWriter, req *http.Request) {
    rw.Header().Set("Content-Type", "application/json")

    encoder := json.NewEncoder(rw)
    encoder.Encode(cities)
}
```

In the `init` method I register `handleIndex` to the `http.DefaultServeMux` which is used by Google's App Engine.

To make sure everything is working as expected I've added a few tests in [`app_test.go`](https://github.com/pjvds/go-cities-appengine/blob/master/app_test.go):

``` go
package cities

import (
    "net/http"
    "net/http/httptest"
    "strings"
    "testing"
)

func TestHandleIndexReturnsWithStatusOK(t *testing.T) {
    request, _ := http.NewRequest("GET", "/", nil)
    response := httptest.NewRecorder()

    http.DefaultServeMux.ServeHTTP(response, request)

    if response.Code != http.StatusOK {
        t.Fatalf("Response body did not contain expected %v:\n\tbody: %v", "200", response.Code)
    }
}

func TestHandleIndexContainsAmsterdam(t *testing.T) {
    request, _ := http.NewRequest("GET", "/", nil)
    response := httptest.NewRecorder()

    http.DefaultServeMux.ServeHTTP(response, request)

    body := response.Body.String()
    if !strings.Contains(body, "Amsterdam") {
        t.Fatalf("Response body did not contain expected %v:\n\tbody: %v", "Amsterdam", body)
    }
}
```

## The build pipeline

The build pipeline is defined in [wercker.yml](https://github.com/pjvds/go-cities-appengine/blob/master/wercker.yml). It consists of a step that first sets up a Go workspace and then executes to the go commands to get the dependencies, build and test the application.

``` yaml
box: wercker/golang
build:
  steps:
    - setup-go-workspace

    - script:
        name: Get dependencies
        code: |-
            go get

    - script:
        name: Build
        code: |
            go build

    - script:
        name: Test
        code: |-
            go test

```

## Building at wercker

I've added the application to wercker as described in the [Golang with wercker](http://blog.wercker.com/2013/07/10/Golang-on-wercker.html) blog post.

![screenshot of a green build](/images/posts/app-engine-go/app-build-at-wercker.png)

## Creating the project on Google's App Engine

Now I need an application on Google App Engine to deploy to. Creating one is done one by [loging in to Google App Engine](https://cloud.google.com/products/app-engine) and it asks me to create a application:

![create app engine project](/images/posts/app-engine-go/create-app-engine-app.png)

## Add App.yaml App Engine configuration

Google App Engine SDK reads its information from `app.yaml`. I add a new `app.yaml` file to the root of my project that defines all needed properties:

``` yaml
application: go-cities
version: 1
runtime: go
api_version: go1

handlers:
- url: /.*
  script: _go_app
```

## Add a deploy target on wercker

After I created an application on Google App Engine, I log in to wercker and navigate to the settings tab of my application to add a custom deploy target.

![create custom deploy target](/images/posts/app-engine-go/add-custom-deploy-target.png)

I've named it `app-engine` and enable auto deploy for the `master` branch. This means that wercker will automatically trigger a deploy whenever a build succeeds from the `master` branch.

I add an environment variable `APP_ENGINE_PASS` to the deploy target so that I can use it from the deployment pipeline. I mark it as protected to prevent the password value to be available via the user interface and being written to the deploy log.

![add password variable](/images/posts/app-engine-go/add-password-variable.png)

_note: this is not my real password... duh!_

## Define the deploy pipeline

In the previous step I've added a deploy target, which can best be explained as a named configuration context that will be available in the deployment pipeline. Now it is time to add the actual deploy pipeline steps. I add the following to my `wercker.yml`:

``` yaml
deploy:
  steps:
    - setup-go-workspace
    - pjvds/go-appengine-deploy:
        email: pj@wercker.com
        password: $APP_ENGINE_PASS
```

Now the deployment pipeline will execute two steps. The [`setup-go-workspace`](https://app.wercker.com/#applications/51fa5e6ba4037f7171000f75/tab/details) will setup a go workspace and the [`pjvds/go-appengine-deploy`](https://app.wercker.com/#applications/52177327e36a64ff11002960/tab/details) will do the actual deployment for us. I set the email option to `pj@wercker.com` and the password option to the `APP_ENGINE_PASS` environment variable that I have added to the deploy target in the previous step.

## Time to (auto) deploy!

I push the `wercker.yml` change to [GitHub](https://github.com/pjvds/go-cities-appengine) and watch wercker build my project. Since I've added a deploy target with auto deploy enabled for the master branch, wercker will trigger a deploy automatically.

![deploying...](/images/posts/app-engine-go/deploying.png)

## Done!

The API is now running at [cities-api.appspot.com](http://cities-api.appspot.com).

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.
