---
title: Microsoft Windows and .NET launched in wercker-labs
date: 2013-10-25
tags: windows, .net, dotnet, c#, azure
author: Wouter Mooij
published: false
gravatarhash: 37936f9de593de18c525a8e2f5707834
---
<h4 class="subheader">
I am very pleased to announce that we're launching Microsoft Windows support at wercker! In this post I quickly want to share the details of our environment. Use it and let us know if you like it!
</h4>

![wercker loves windows](/images/posts/dart/wercker+windows.jpg)


READMORE


## Introducing wercker-labs

At wercker we're constantly upgrading our software to better suit your needs. In accordance with Contiuous Delivery, we like to keep our changes small and fast.
But sometimes we need some more feedback before we can integrate new features into wercker. We've created wercker-labs as label for these new features.
Anything released under wercker-labs can change very fast, but you can expect the same level of support that you're accustomed to.

We need you for your feedback, so please send us your issues, rants, love, or better yet: pull requests!

## wercker-labs/windows box

We're very happy that the first box that we're launching is a Microsoft Windows Server 2008 R2 SP1 Datacenter 64-bit box with SQL 2008 R2 Web. 

On the box we've installed a lot of software to help you build, test and deploy .NET solutions. You can check the details page for an overview: https://app.wercker.com/#applications/526138951c260dab440001a8/tab/details

If you need anything else, please let us know!

## Steps for wercker-labs/windows

Our goal is to make building, testing and deploy .NET solutions as easy as possible. Therefore we've created some steps to help you.


### wercker-labs/nuget

If you're using NuGet, you can use `wercker-labs/nuget` to install your packages.

Check the details page for more information: https://app.wercker.com/#applications/52623205f4a7b1d05a00240b/tab/details

### wercker-labs/msbuild

To build your solution, you can use MsBuild and the `wercker-labs/msbuild` step.

Check the details page for more information: https://app.wercker.com/#applications/526231a7f4a7b1d05a002397/tab/details

### wercker-labs/dotnet-test

We've created the `wercker-labs/dotnet-test` to support as many testing libraries as possible. Gallio is used as testrunner.

Check the details page for more information: https://app.wercker.com/#applications/526231e0f4a7b1d05a0023dc/tab/details

### wercker-labs/azure-ftp-deploy

If you want to deploy your code to Windows Azure, you can use the `wercker-labs/azure-ftp-deploy` step. It supports the simple use case of uploading your compiled code to Windows Azure over FTP.

Check the details page for more information: https://app.wercker.com/#applications/52691aa3d64894582a010e31/tab/details

## Default wercker.yml

Wercker analyses your code to determine what kind of application you are working on.
If there is a .sln file in the root of your repository, then this default wercker.yml will be used.

```
box: wercker-labs/windows
build:
  steps:
    - wercker-labs/nuget
    - wercker-labs/msbuild
    - wercker-labs/dotnet-test
```


## A complete example

To show you how easy it is to build, test and deploy a .NET solution, we've created a simple MVC application.

### SimpleAppNet

The solution consists of two projects. SimpleApp.Web is a default MVC application and SimpleApp.UnitTests contains the default unit tests for SimpleApp.Web.

You can see the code (or fork it!) here: https://github.com/wercker-labs/dotnet-simple-app

### Build

If we just want to build our code, we can use this wercker.yml.

```
box: wercker-labs/windows
build:
  steps:
    - wercker-labs/msbuild
```

The `wercker-labs/msbuild` steps will detect the solution in the root of the repository and will build it with the default configuration.


### Test

To add our tests to wercker, we need to add `wercker-labs/dotnet-test` to wercker.yml.


```
box: wercker-labs/windows
build:
  steps:
    - wercker-labs/msbuild
    - wercker-labs/dotnet-test
```

The `wercker-labs/dotnet-test` step will scan all projects and when a project with "test" in the name is found, it will run it's tests.


### Windows Azure

We want to deploy to Windows Azure and before we can do that, we need to create a website and get the publish profile.

* Go to Windows Azure https://manage.windowsazure.com. 
* Create a new web site. 
* On the dashboard, download the publish profile. 
* Open the file in your favorite text editor.
* Find the publishProfile element with publishMethod="FTP". 
* Get publishUrl, userName and userPWD.

Now we've setup Windows Azure and got the information we need to deploy to it.


### Deploy with wercker

To deploy you need to create a deploy target on wercker. Go to the Settings tab of your application. Use "Add deploy target" and "Custom deploy" to create one.
Name it anyway you like it, eg staging or production.
To keep your password save, we create a new variable by clicking "+ Add new variable". Use "AZURE_PASSWORD" as name, copy the content of "userPWD" you found in your publish profile, check "protected" and click "ok". Now click "Save" to save your deploy target.

By default wercker saves all files to use for deployment. But we only need the files from the SimpleApp.Web directory. We create an extra step in the build pipeline to make this work. We're using rsync: `rsync -avz SimpleApp.Web/ $WERCKER_OUTPUT_DIR/`.

Now all you need to do is add `wercker-labs/azure-ftp-deploy` to your deploy pipeline and configure the url, username and password.

When you've done this, your wercker.yml should look something like this:

```
box: wercker-labs/windows
build:
  steps:
    - wercker-labs/msbuild
    - wercker-labs/dotnet-test
    - script:
        name: Copy to output
        code: rsync -avz SimpleApp.Web/ $WERCKER_OUTPUT_DIR/
deploy:
  steps:
    - wercker-labs/azure-ftp-deploy:
        publish-url: ftp://waws-prod-blu-003.ftp.azurewebsites.windows.net/site/wwwroot
        username: test\\\$test
        password: $AZURE_PASSWORD
```

It's important to note that because we're using environment variables, you need to escape the `\` and `$` characters in the username by prefixing it with an `\`.

If you pushed your changes, you van deploy this build using the deploy button.

## Resources

Here are the resources related to this sample project:

* [Sample project source code at GitHub](https://github.com/wercker-labs/dotnet-simple-app)
* [Windows box at wercker directory](https://app.wercker.com/#applications/526138951c260dab440001a8/tab/details)
* [MsBuild step at wercker directory](https://app.wercker.com/#applications/526231a7f4a7b1d05a002397/tab/details)
* [NuGet step at wercker directory](https://app.wercker.com/#applications/52623205f4a7b1d05a00240b/tab/details)
* [.NET test step at wercker directory](https://app.wercker.com/#applications/526231e0f4a7b1d05a0023dc/tab/details)
* [Windows Azure FTP step at wercker directory](https://app.wercker.com/#applications/52691aa3d64894582a010e31/tab/details)

## Earn some stickers of your own!

Let us know about the applications you build with wercker. Don't forget to tweet out a screenshot of your first green build with **#wercker** and we'll send you some [@wercker](http://twitter.com/wercker) stickers.

Thanks for tuning in, and there will be more posts next week! Follow us on [twitter](http://twitter.com/wercker) as well to stay in the loop.
