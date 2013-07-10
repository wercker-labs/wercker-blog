---
title: Golang with wercker
date: 2013-07-10
tags: golang, structure
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
<a href="https://app.wercker.com/#pjvds">Pieter Joost</a> our community manager and engineer has added support for Google's Go programming language (golang).
In this post we'll briefly discuss setting up your golang project!
</h4>

![image](http://f.cl.ly/items/2x3L2l1R2o1P1c0n3R3c/b91af6a4e95911e2a54722000a9f1597_7.jpg)

READMORE

## Creating our application

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/). We've open sourced the code for this application on [GitHub](https://github.com/mies/go-http-sample).

You can find this application on wercker with its current build status:

[![Wercker status](https://app.wercker.com/status/91c291356b4383d2f7fe3df71fe99314/m)](https://app.wercker.com/project/bykey/91c291356b4383d2f7fe3df71fe99314)

We'll be creating a simple web application in go, backed by [Gorilla's Mux](http://www.gorillatoolkit.org/pkg/mux), a request and routing dispatcher for go.
Mux is ideal for writing small API's and the Gorilla Web Toolkit has a wealth of libraries and packages for writing go web applications.

Create a file called `main.go` with the following contents:

``` go
package main

import "encoding/json"
import "net/http"
import "log"
import "os"

import "github.com/gorilla/mux"

func router() *mux.Router {
    router := mux.NewRouter()
    router.HandleFunc("/", IndexHandler).Methods("GET")
    return router
}

func routerHandler(router *mux.Router) http.HandlerFunc {
    return func(res http.ResponseWriter, req *http.Request) {
        router.ServeHTTP(res, req)
    }
}

func IndexHandler(res http.ResponseWriter, req *http.Request) {
    data, _ := json.Marshal("{'hello':'wercker!'}")
    res.Header().Set("Content-Type", "application/json; charset=utf-8")
    res.Write(data)
}

func main() {
    handler := routerHandler(router())
    err := http.ListenAndServe(":"+os.Getenv("PORT"), handler)
    if err != nil {
        log.Fatal("ListenAndServe: ", err)
    }
}

```

Our API creates a [Mux Router](http://www.gorillatoolkit.org/pkg/mux#Router) and a handler for the `/`, home route. We then start the HTTP server and listen on the `$PORT` environment variable, which is convenient for when we want to deploy this application for instance to Heroku that leverages this environment variable.

This API outputs `{'hello': 'wercker!'}` in JSON which we marshal using the go standard [JSON library](http://golang.org/pkg/encoding/json/).

## Writing our unit test

Next we will write a simple unit test for our API. Create a file called `main_test.go` with the following contents:

``` go
package main

import (
    "net/http"
    "net/http/httptest"
    "testing"
)

func TestHandleIndexReturnsWithStatusOK(t *testing.T) {
    request, _ := http.NewRequest("GET", "/", nil)
    response := httptest.NewRecorder()

    IndexHandler(response, request)

    if response.Code != http.StatusOK {
        t.Fatalf("Non-expected status code%v:\n\tbody: %v", "200", response.Code)
    }
}
```

This unittest uses the [httptest librariy](http://golang.org/pkg/net/http/httptest/) to create a request to our `IndexHandler`. We simply test if the **status code** returns a **200OK**.

Now push this code to GitHub or Bitbucket:

``` bash
git init
git add .
git commit -am 'init'
```

## Adding our app to wercker

Now we are ready to test and build our application with wercker. Add your project to wercker either via the [command line](http://devcenter.wercker.com/articles/gettingstarted/cli.html) [interface](http://devcenter.wercker.com/articles/cli/) or via the [web interface](http://devcenter.wercker.com/articles/gettingstarted/web.html).

Now create a file called `wercker.yml`. The [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file describes your build and deployment pipeline on wercker. For this simple application we just need to let wercker now, that you have a `golang` project and want to make use of the golang box. Add the following line to your `wercker.yml` file:

``` yaml
box: pjvds/golang
```

You can read more about the `werkcer.yml` file on our [dev center](http://devcenter.wercker.com/articles/werckeryml/).

By default the golang box does the following:

* fetch your dependencies (in this case the Mux library)
* compile and build your application
* test your application

These items are sufficient to test and build our application, so we do not need add anything more to our **wercker.yml** file with regards to build steps.

Add the **wercker.yml** file to your git repository and push it.

``` bash
git add wercker.yml
git commit -am 'added wercker.yml'
git push origin master
```

This will trigger a build on wercker which should pass.

![image](http://f.cl.ly/items/0j0L0Q0n143Q0q3n430R/Screen%20Shot%202013-07-10%20at%202.34.35%20PM.png)

Congrats, you've now built your golang application on wercker!

We'll be posting an article on golang and deployment later on, so stay tuned.

## Earn some stickers of your own!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.

