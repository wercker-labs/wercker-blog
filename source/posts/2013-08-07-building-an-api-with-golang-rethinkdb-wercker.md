---
title: Building an API with Golang, RethinkDB and wercker
date: 2013-08-07
tags: rethinkdb, golang
author: Micha Hernandez van Leuffen
gravatarhash: d4b19718f9748779d7cf18c6303dc17f
---

<h4 class="subheader">
In a <a href="http://blog.wercker.com/2013/07/23/Building-your-own-box-with-Bash.html">previous article</a> we've explained how to build your own <a href="http://devcenter.wercker.com/articles/boxes/">wercker box</a> provisioned with <a href="http://rethinkdb.com">RethinkDB</a>, an open source distributed database, which is quite awesome.
</h4>

![image](http://f.cl.ly/items/330P0J3G2d0S3Y2q2U0Z/Image%202013.08.07%204%3A15%3A36%20PM.jpeg)

In this tutorial we are going to explain how to build a small API in Go, backed by RethinkDB. We are going to leverage the excellent RethinkDB [driver for Go](https://github.com/christopherhesse/rethinkgo) by [Christopher Hesse](https://twitter.com/christophrhesse).

![image](http://f.cl.ly/items/133A2T2C003n253o1e1i/RethinkDb.png)

Signing up for wercker is [free and easy](https://app.wercker.com/users/new/). You can check out the code for this tutorial [here](https://github.com/mies/getting-started-golang-rethinkdb) and view its status on wercker [here](https://app.wercker.com/#applications/5202373e3ca948bc5a004cd1)

[![wercker status](https://app.wercker.com/status/16fb140467c41c40386174196481943f/m "wercker status")](https://app.wercker.com/project/bykey/16fb140467c41c40386174196481943f)

READMORE


## Getting started

First, make sure you have RethinkDB installed and of course have a [wercker account](https://app.wercker.com/users/new/). 
Read the [installation instructions](http://rethinkdb.com/docs/install/) on their recently revamped [documentation section](http://rethinkdb.com/docs/).

The API we will be building will store and retrieve bookmarks. It is extremely basic but should provide enough context for your own applications.

Let's start out by writing our API. Create a file called `main.go` with the following contents:

``` go
package main

import (
	"log"
	"os"
	"net/http"
	"encoding/json"
	r "github.com/christopherhesse/rethinkgo"
)

var sessionArray []*r.Session

type Bookmark struct {
	Title string
	Url	  string
}

func initDb() {
	session, err := r.Connect(os.Getenv("WERCKER_RETHINKDB_URL"), "gettingstarted")
	if err != nil {
		log.Fatal(err)
		return
	}

	err = r.DbCreate("gettingstarted").Run(session).Exec()
	if err != nil {
	  log.Println(err)
    }

	err = r.TableCreate("bookmarks").Run(session).Exec()
    if err != nil {
	  log.Println(err)
    }

	sessionArray = append(sessionArray, session)
}

func main() {

	initDb()

	http.HandleFunc("/", handleIndex)
	http.HandleFunc("/new", insertBookmark)

	err := http.ListenAndServe(":5000", nil)
	if err != nil {
		log.Fatal("Error: %v", err)
	}
}

func insertBookmark(res http.ResponseWriter, req *http.Request) {
	session := sessionArray[0]

	b := new(Bookmark)
	json.NewDecoder(req.Body).Decode(b)

	var response r.WriteResponse

	err := r.Table("bookmarks").Insert(b).Run(session).One(&response)
	if err != nil {
		log.Fatal(err)
		return
	}
	data, _ := json.Marshal("{'bookmark':'saved'}")
	res.Header().Set("Content-Type", "application/json; charset=utf-8")
	res.Write(data)
}

func handleIndex(res http.ResponseWriter, req *http.Request) {
	session := sessionArray[0]
	var response []Bookmark

	err := r.Table("bookmarks").Run(session).All(&response)
	if err != nil {
		log.Fatal(err)
	}

	data, _ := json.Marshal(response)

	res.Header().Set("Content-Type", "application/json")
	res.Write(data)
}

```

We do the usual imports for **logging**, **os** function calls, **http** and **json**. We also import the previously mentioned [go RethinkDB driver](https://github.com/christopherhesse/rethinkgo) and make it available as `r`.

We create an array of `Sessions` that will just hold one connection to RethinkDB in this example, but could also hold a pool of connections.

Next, we create a struct for our Bookmark type consisting of a `title` and a `url`.

Using the `initDB` function we actualy set up our connection to RethinkDB and add our session to the previously mentioned `sessionArray`. We also create the database called `gettingstarted` with a table called `Bookmarks`. We log but don't fail if these creation steps do not succeed as both the database and table might already exist. As we're using [wercker](http://wercker.com) to build, and eventually, deploy our app, we only connect to the RethinkDB instance running on wercker using the `WERCKER_RETHINKDB_URL` environment variable. For your own apps you might want to setup the connection strings depending on the **dev**, **test**, **staging** or **production** environments your code is running in.

Now we're getting to the heart of our API! The **main** function calls the `initDb()` to set up our connection. We have two url routes; the index (`/`) will return the stored bookmarks as JSON. and the `/new` method will store a newly created bookmark through an HTTP POST call.

We listen on port number `5000`, which we might want to export as a `PORT` environment variable and read using `os.Getenv`, in true [12factor](http://12factor.net) fashion. We do not do this in this example though.

The `insertBookmark` function creates a new Bookmark depending on the body of the POST request (which expects a **title** and a **url**). We then return a JSON response with `{'bookmark':'saved'}`.

The `handleIndex` function retrieves the stored bookmarks from RethinkDB which we then return as JSON.

## Writing our test

We are now ready to test our API. Create a file called `main_test.go` with the following contents:

``` go
package main

import (
	"net/http"
	"net/http/httptest"
	"strings"
	"testing"
	"encoding/json"
	"bytes"
)

func init() {
	initDb()
}

func TestHandleIndexReturnsWithStatusOK(t *testing.T) {
	request, _ := http.NewRequest("GET", "/", nil)
	response := httptest.NewRecorder()

	handleIndex(response, request)

	if response.Code != http.StatusOK {
		t.Fatalf("Response body did not contain expected %v:\n\tbody: %v", "200", response.Code)
	}
}

func TestHandInsertBookmarkWithStatusOK(t *testing.T) {
	bookmark := Bookmark{"wercker", "http://wercker.com"}
	
	b, err := json.Marshal(bookmark)
	if err != nil {
		t.Fatalf("Unable to marshal Bookmark")
	}

	request, _ := http.NewRequest("POST", "/new", bytes.NewReader(b))
	response := httptest.NewRecorder()

	insertBookmark(response, request)
	
	body := response.Body.String()	
	if !strings.Contains(body, "{'bookmark':'saved'}") {
		t.Fatalf("Response body did not contain expected %v:\n\tbody: %v", "San Francisco", body)
	}
}

func TestHandleIndexReturnsJSON(t *testing.T) {
	request, _ := http.NewRequest("GET", "/", nil)
	response := httptest.NewRecorder()
	
	handleIndex(response, request)
	
	ct := response.HeaderMap["Content-Type"][0]
	if !strings.EqualFold(ct, "application/json") {
		t.Fatalf("Content-Type does not equal 'application/json'")
	}
}
```

In the test we also set up our session to RethinkDB by calling `initDb` in the `init` function. We have three test cases. In `TestHandleIndexReturnsWithStatusOK` we check if calling the root url (`/`) returns a 200OK status code. Next, we check if we can insert a bookmark and get the `{'bookmark':'saved'}` JSON response back. Our final test case checks if we get a JSON response back from the root url which should return our stored bookmarks (in JSON).

## Adding our project to wercker

We are now ready to build our application using wercker (we will handle deployment in another post). Make sure your project is either stored in [GitHub](http://github.com) or [Bitbucket](http://bitbucket.org/). Next add your project to wercker. You can check the guides on our [dev center](http://devcenter.wercker.com) that explain to add your application either via the [web interface](http://devcenter.wercker.com/articles/gettingstarted/web.html) or the [command line interface](http://devcenter.wercker.com/articles/gettingstarted/cli.html).

The **add application wizard** will present you with a default [wercker.yml](http://devcenter.wercker.com/articles/werckeryml/) file to set up your build pipeline for Go projects on wercker. Create a file called `wercker.yml` that looks as follows:

``` yaml
box: wercker/golang
# Services
services:
    - mies/rethinkdb
# Build definition
build:
  # The steps that will be executed on build
  steps:
    # Sets the go workspace and places you package
    # at the right place in the workspace tree
    - setup-go-workspace

    # Gets the dependencies
    - script:
        name: go get
        code: |
          cd $WERCKER_SOURCE_DIR
          go version
          go get ./...

    # Build the project
    - script:
        name: go build
        code: |
          go build ./...

    # Test the project
    - script:
        name: go test
        code: |
          go test ./...

```
We run our code inside the `wercker/golang` [box](https://app.wercker.com/#applications/51ad0329c67e056078000876/tab/details) and leverage the [RethinkDB service box](https://app.wercker.com/#applications/51cc29f604a2788145000b67/tab/details). We also make use of the [setup-go-workspace](https://app.wercker.com/#applications/51fa5e6ba4037f7171000f75/tab/details) [step](http://devcenter.wercker.com/articles/steps/) that adds the project to the Go workspace directory hierarchy.

The build pipeline fetches dependencies, builds our code and runs our tests. 

Now, add your files to git and push them out to your version control system of choice. 

``` bash
git add .
git commit -am 'init'
git push origin master
```

Each time you do a `git push` wercker will build your project which looks as follows:

![image](http://f.cl.ly/items/1w3U2m3I2P2J3r28372i/Screen%20Shot%202013-08-07%20at%203.55.06%20PM.png)

You can read more on building and deploying golang apps with wercker on the [golang section](http://devcenter.wercker.com/articles/languages/go.html) on our [dev center](http://devcenter.wercker.com).

In a subsequent article we will discuss deploying your golang + RethinkDB application with wercker!

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.
