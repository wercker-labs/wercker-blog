---
title: Code coverage with gocov and wercker
date: 2013-08-05
tags: gocov, golang, codecoverage
author: Pieter Joost van de Sande
gravatarhash: 5864d682bb0da7bedf31601e4e3172e7
published: false
---
<h4 class="subheader">
I have automated the build and deployment pipeline for my blog. For every push it generates my site, which validates the content and tests it for known depricates. It generates two different versions of the site. One for staging and one for production. When I push from the `staging` it automaticly gets deployed to my staging environment. The same goes for the `master`, which automaticly gets deployed to my production environment.
</h4>

READMORE

Although I am very happy on the pipeline I've got so far, I still find myself refreshing [born2code.net](http://born2code.net) after I pushing to the master branch to make sure everything it still running after the deployment. I fixed this by adding a few simple post deployment tests that now do this job for me.

## Post deploy smoke test

My deployment consists of synchronizing the static generated website from the build pipeline to an Amazon S3 bucket. Here is how the deployment pipeline defined in my [wercker.yml]() looks like:

``` yaml
deploy:
  steps:
    - s3sync:
        key-id: $KEY
        key-secret: $SECRET
        bucket-url: $BUCKET
        source-dir: $SOURCE
```

The environment variables are defined in the deployment targets at wercker. Staging has other values then production.

Although I can think of some quite impressive post deploy tests, the most effective one is to see if the site is still running and that the html that is returned by the server is not from an error page.

## post-deploy-tests.sh

I start by adding an empty shell script `post-deploy-tests.sh` file to my repository:

``` bash
touch post-deploy-tests.sh
```

The first step is to make a request to the site and make sure the status code is still 200.

The status code can be retrieved with `curl` and it's `--write-out` option. Here is an example that stores the status code of a request in `status_code`:

``` bash
status_code=$(curl --write-out %{http_code} "$SITE_URL")
```

I use the wercker essentials methods to log failures.

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

Now I can assert the response content as well. I simply grab the title from the html with `awk` to make sure that one is correct.

``` bash
# Check title is correct
title=$(awk -vRS="</title>" '/<title>/{gsub(/.*<title>|\n+/,"");print;exit}' site-content.html)
if [ $title != 'born2code.net' ]; then
  fail "Unexpected title '$title' for $SITE_URL";
fi
```

I finalize the file by adding some extra logging, including a success at the end.

``` bash
# Check status is 200
status_code=$(curl --write-out %{http_code} --silent --output "site-content.html" "$SITE_URL")
if [ $status_code -ne 200 ]; then
  fail "Deploy failure: site returned $status_code, 200 was expected";
else
  info "Status code is correct: $status_code";
fi

# Check title is correct
title=$(awk -vRS="</title>" '/<title>/{gsub(/.*<title>|\n+/,"");print;exit}' site-content.html)
if [ $title != 'born2code.net' ]; then
  fail "Unexpected title '$title' for $SITE_URL";
else
  info "Title is correct: $title";
fi

# We reached the end of the script
success 'Post deploy tests succeeded'
```

## Use the script from wercker.yml

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
