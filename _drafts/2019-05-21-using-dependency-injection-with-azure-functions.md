---
layout: post
title: Using Dependency Injection with C# .NET Azure Functions (with example)
categories: [azure, csharp, dotnet, serverless, functions]
tags: [csharp, serverless, rest, api, azure, dotnet, functions, injection]
description: In this post we will look at the recently released dependency injection support for Azure Functions
comments: true
---

The code for this example can be found [here](https://github.com/tomling/azure-functions-examples/tree/master/AzureFunctionsExamples/FunctionsDependencyInjection)

## What is dependency injection

Dependency injection (DI) is a software design pattern that allows developers to write loosely coupled code. This is accomplished by specifying which objects a class requires at run time, rather than implementing a concrete dependency. By using DI injection in your applications you can ensure your code base is maintainable, testable and easy to update.

## Implementing dependency injection in Azure Functions

Microsoft have recently released the Microsoft.Azure.Functions.Extensions NuGet package (at the time of writing version 1.0.0). This package adds native support for dependency injection by building upon the existing ASP.NET Core Dependency Injection features. In this example we will use this package to implement DI into a simple REST trigger C# function.

First step is to create a HTTP function, this can be achieved by either through Visual Studio or using the Azure Function CLI

To use the Azure Function CLI you will need the Azure Functions Core Tools which you can install through npm

```text
npm i -g azure-functions-core-tools --unsafe-perm true
```

First you will need to create the function project

```text
func init <PROJECT NAME> --worker-runtime dotnet
```

Then change directory to the newly created project and run the following to create the function.

```text
func new --name <FUNCTION NAME> --template "HttpTrigger"
```

After creating the project you will need to add a reference to the Microsoft.Azure.Functions.Extensions package and update to the latest version of the function SDK. You can do this via using the dotnet CLI.

```text
dotnet add package Microsoft.Azure.Functions.Extensions
dotnet add package Microsoft.NET.Sdk.Functions
```

First we are going to create the service that we will inject into our function. In this example we will be using a simple class that returns a greeting message. Create the following files (IGreeter.cs and Greeter.cs) in your function project.

<script src="https://gist.github.com/tomling/e1c62bd68f306738af7cfe6e34573f0f.js"></script>

<script src="https://gist.github.com/tomling/fa39e58fa099ce82c5b71769a29d4847.js"></script>

Next we will need to setup the Startup class which will be used to register our service in the application. There are three main types of service lifetimes you can choose from when registering a service.

**Transient -** An instance of your service is created every time they are requested from the service container. This is recommended for lightweight, stateless services.

**Scoped -** Service is created once per client request (connection).

**Singleton -** Singleton services are created the first time they're request, then every subsequent request uses the same instance.

In this example I will use the transient service lifetime, as I want to create an instance of the Greeter every time my function is invoked.

<script src="https://gist.github.com/tomling/aaa882566b8a1d6861ccf023d6783d41.js"></script>

Finally we need to adapt our HTTP function to use our Greeter service by using constructor injection, this means that we will also need to convert our function to be non-static.

<script src="https://gist.github.com/tomling/b2d73278de777419124ac72f3707f0cc.js"></script>

To test our function, run the following command using the function CLI and navigate to the link provided in the terminal in a browser.

```text
func host start
```

If you see a message with "Greetings from Greeter!" then that means dependency injection for our service has been configured correctly. Nice one!

## Wrap up

Using the dependency injection features provided with the Microsoft.Azure.Functions.Extensions package means that we can easily produce code that is maintainable, testable and easy to refactor. If you require any more additional information about implementing DI into Azure functions then check out the official Microsoft page [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-dependency-injection).
