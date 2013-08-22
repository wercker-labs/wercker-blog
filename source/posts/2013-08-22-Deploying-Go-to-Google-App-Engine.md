---
title: Deploying Go to Google App Engine
date: 2013-08-22
tags: deployment, golang, AppEngine
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
published: false
---

## The application

I've created a small [application](https://github.com/pjvds/go-cities-appengine/) in [Go](http://golang.org) that prints a list of cities on request. The application logic can be found in [`app.go`](https://github.com/pjvds/go-cities-appengine/blob/master/app.go):

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

In the `init` method I register `handleIndex` to the `http.DefaultServeMux` which is used by Google's AppEngine.

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

The build pipeline is defined in [wercker.yml](https://github.com/pjvds/go-cities-appengine/blob/master/wercker.yml). It defines that

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

    - script:
        name: Copy output
        code: |-
          rsync -avz "$WERCKER_SOURCE_DIR/" "$WERCKER_OUTPUT_DIR"
```

## Building at wercker

I've added the application to wercker as described in the [Golang with wercker](http://blog.wercker.com/2013/07/10/Golang-on-wercker.html) blogpost.

![screenshot of a green build]()

## Creating the project at Google's AppEngine

Now I need an application at Google App Engine to deploy to. I create one by [loging in to Google App Engine](https://cloud.google.com/products/app-engine) and it asks me to create a application:

![create app engine project]()

## Add App.yaml App Engine configuration

Google App Engine SDK reads it's information from `app.yaml`. I add a new `app.yaml` file to the root of my project that defines all needed properties:

``` yaml
application: go-cities
version: 1
runtime: go
api_version: go1

handlers:
- url: /.*
  script: _go_app
```

## Create an deployment target at wercker

After I created an application at Google App Engine, I log in to wercker and navigate to the settings tab of my application to add an custom deploy target.

![create custom deploy target]()

I name it `app-engine` and enable auto deploy for the `master` branch. Which means that wercker will automatically trigger an deployment whenever a build succeeds from the `master` branch.

I add an environment variable `APP_ENGINE_PASS` to the deploy target so that I can use it from the deployment pipeline. I mark it as protected to prevent the password value to be available via the user interface and will not be written to the deploy log.

![add password variable]()

_note: this is not my real password... duh!

## Define the deploy pipeline

In the previous step I added a deploy target, which can best be explained as an named configuration context that will be available in the deployment pipeline. Now it is time to add the actual deployment pipeline steps. I add the following to my `wercker.yml`:

``` yaml
deploy:
  steps:
    - setup-go-workspace
    - pjvds/appengine-deploy:
        email: pj@wercker.com
        password: $APP_ENGINE_PASS
```

Now the deployment pipeline will execute two steps. The [`setup-go-workspace`](https://app.wercker.com/#applications/51fa5e6ba4037f7171000f75/tab/details) will step a go workspace and the [`pjvds/appengine-deploy`](https://app.wercker.com/#applications/520cc5498a20a26245010fb9/tab/details) will do the actual deployment for us. I set the email option to `pj@wercker.com` and the password option to the `APP_ENGINE_PASS` environment variable that I have added to the deploy target in the previous step.

## Time to (auto) deploy!

I push the `wercker.yml` change to [GitHub](https://github.com/pjvds/go-cities-appengine) and watch wercker building my project. Since I've added a deploy target with auto deploy enabled for the master branch wercker will trigger a deploy automatically.

![deploying...]()

## Done!

The API is now running at [cities-api.appspot.com](http://cities-api.appspot.com).
