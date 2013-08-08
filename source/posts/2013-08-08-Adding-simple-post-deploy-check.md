---
title: Adding simple Post Deploy checks with wercker
date: 2013-08-08
tags: gocov, golang, codecoverage
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
---

<h4 class="subheader">
I have automated the <a href="http://blog.wercker.com/2013/05/31/simplify-you-jekyll-publishing-process-with-wercker.html">build</a> and <a href="http://blog.wercker.com/2013/07/30/adding-a-staging-environment-to-your-blog.html">deployment</a> pipeline for my personal <a href="http://born2code.net">blog</a>. For every push, wercker generates my site, which validates the content and tests it for known deprecationss. It generates two different versions of the site. One for staging and one for production. When I push from the `staging` branch, it automaticly gets deployed to my staging environment. The same goes for the `master`, which automatically gets deployed to my production environment.
</h4>

![image](http://f.cl.ly/items/1T2B40451f1O3R0d1M2Y/A0284044-4F04-4350-82FB-0E78A59D461C.jpg)

READMORE

Although I am very happy with the pipeline I've got so far, I still find myself refreshing [born2code.net](http://born2code.net) after pushing to the master branch to make sure everything it still running after the deployment. I fixed this by adding a few simple post deployment tests that now do this job for me, all with some simple Bash scripting.

## Post deploy smoke test

My deployment [pipeline](http://devcenter.wercker.com/articles/introduction/pipeline.html) consists of synchronizing the static generated website from the build pipeline to an Amazon S3 bucket. Here is how the deployment pipeline is defined in my [wercker.yml](http://devcenter.wercker.com/articles/introduction/pipeline.html) file:

``` yaml
deploy:
  steps:
    - s3sync:
        key-id: $KEY
        key-secret: $SECRET
        bucket-url: $BUCKET
        source-dir: $SOURCE
```

The environment variables are defined in the deployment targets at wercker. In my case the staging target has other values than production.

Although I can think of some quite impressive post deploy tests, the most effective one is to see if the site is still running, and that the HTML which is returned by the server is not the HTML from an error page.

### post-deploy-tests.sh

I start by adding an empty shell script `post-deploy-tests.sh` file to my repository:

``` bash
touch post-deploy-tests.sh
```

The first step is to make a request to the site and make sure the status code is still **200OK**.

The status code can be retrieved with `curl` and its `--write-out` option. Here is an example that stores the status code of a request in `status_code`:

``` bash
status_code=$(curl --write-out %{http_code} "$SITE_URL")
```

Wercker has several `Bash` methods to log failures.

``` bash
status_code=$(curl --write-out %{http_code} "http://born2code.net")
if [ $status_code -ne 200 ]; then
  fail "Deploy failure: site returned $status_code, 200 was expected";
fi
```

I change the `curl` command to also write the content to a file.

``` bash
status_code=$(curl --write-out %{http_code} --silent --output "site-content.html" "http://born2code.net")
if [ $status_code -ne 200 ]; then
  fail "Deploy failure: site returned $status_code, 200 was expected";
fi
```

Now I can assert the response body as well. I simply grab the title from the html with `awk` to make sure it is the correct title.

``` bash
### Check title is correct
title=$(awk -vRS="</title>" '/<title>/{gsub(/.*<title>|\n+/,"");print;exit}' site-content.html)
if [ $title != 'born2code.net' ]; then
  fail "Unexpected title '$title' for $SITE_URL";
fi
```

I finalize the file by adding some extra logging, including a success at the end. You can find my `post-deploy-tests.sh ` script below:

``` bash
### Check status is 200
status_code=$(curl --write-out %{http_code} --silent --output "site-content.html" "$SITE_URL")
if [ $status_code -ne 200 ]; then
  fail "Deploy failure: site returned $status_code, 200 was expected";
else
  info "Status code is correct: $status_code";
fi

### Check title is correct
title=$(awk -vRS="</title>" '/<title>/{gsub(/.*<title>|\n+/,"");print;exit}' site-content.html)
if [ $title != 'born2code.net' ]; then
  fail "Unexpected title '$title' for $SITE_URL";
else
  info "Title is correct: $title";
fi

### We reached the end of the script
success 'Post deploy tests succeeded'
```

## Use the script tag in your wercker.yml

I add the script execution at the end of my deploy pipeline by sourcing it. My complete deployment pipeline now looks like this:

``` yaml
deploy:
  steps:
    - s3sync:
        key-id: $KEY
        key-secret: $SECRET
        bucket-url: $BUCKET
        source-dir: $SOURCE

    - script:
        name: Post deploy tests
        code: source post-deploy-tests.sh
```

## Earn some stickers!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.
