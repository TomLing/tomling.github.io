---
layout: post
title: Azure Functions 2.0 HTTP routing options
categories: [azure,csharp,dotnet,serverless,functions]
tags: [csharp,serverless,api,azure,dotnet,functions,routing]
description: In this post I will cover the various HTTP routing configurations when using Azure Functions.
comments: true
---

## What is routing

Routing refers to the way an application responds to a client request to a particular endpoint address and specific HTTP request method (GET, POST etc). In the context of Azure Functions, the route defines which function is going to respond to a HTTP request.

## Azure Functions Default Routing

To demonstrate the various routing options, I will be using the default Azure Functions 2.0 (.NET Core) HTTP trigger template which looks a like this, (at the time of writing).

{% highlight csharp %}
[FunctionName("NameFunction")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
    ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string name = req.Query["name"];

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;

    return name != null
        ? (ActionResult)new OkObjectResult($"Hello, {name}")
        : new BadRequestObjectResult("Please pass a name on the query string or in the request body");
}
{% endhighlight %}

By running this locally you can see that the default URL in order to call this function is **_http://localhost:7071/api/NameFunction_**
This URL is comprised of a route prefix of api, then the name of your function, in this instance 'NameFunction'.

![Default Routing]({{ "/assets/media/2019-01-24/defaultroute.PNG" | absolute_url }})

*Running the app exposes the default URL*

Navigating to this URL in the browser supplied with a parameter with your name yields the following result.

![Default Route Browser Result]({{ "/assets/media/2019-01-24/defaultroutebrowser.PNG" | absolute_url }})

## Changing the route prefix with host.json

To change the app's default routing of **_/api/_** the **host.json** file will need to be modified. This is a file that contains global configuration options that affect all functions for a given function app.

To modify the route prefix, amend the **host.json** file to look like the following

{% highlight json %}
{
  "version": "2.0",
  "extensions": {
    "http": {
      "routePrefix": "Name"

    }
  }
}
{% endhighlight %}

This will turn **_http://localhost:7071/api/NameFunction_** into **_http://localhost:7071/Name/NameFunction_**.

In order to remove the route prefix completely modify the routePrefix section of the **host.json** to match the following.

{% highlight json %}
{
  "version": "2.0",
  "extensions": {
    "http": {
      "routePrefix": ""

    }
  }
}
{% endhighlight %}

This will turn **_http://localhost:7071/api/NameFunction_** into **_http://localhost:7071/NameFunction_**.

Removing the route prefix, I can still call the API as I normally would by dropping it from the URL.

![Remove Route Prefix Browser Result]({{ "/assets/media/2019-01-24/defaultroutebrowserwithoutprefix.PNG" | absolute_url }})

## Define the route in the function header

Routing can also be defined by modifying the HttpTrigger attribution in the header of the function, by default this is set to null.
By changing this value to 'GetName' will turn the URL into **_http://localhost:7071/api/GetName_**.

{% highlight csharp %}
[FunctionName("NameFunction")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = "GetName")] HttpRequest req,
    ILogger log)
{ ...
{% endhighlight %}

The route can also be changed to a blank string. This will change the URL into **_http://localhost:7071/api/_**

## Adding parameters to function routes

To add parameters to the route of your function, you will need to add the parameter name in curly braces in the route property of the HttpTrigger attribute, in addition adding it into the method parameters.

To demonstrate this refactor the default HTTP trigger code to match the following

{% highlight csharp %}
[FunctionName("NameFunction")]
public static IActionResult Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = "GetName/{name}")] HttpRequest req,
    string name,
    ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    return new OkObjectResult($"Hello, {name}");
}
{% endhighlight %}

This will change the URL to the following **_http://localhost:7071/api/GetName/{name}_**

Now instead of supplying the name via a URL parameter, I can instead supply it using a route parameter.

![Route Parameter]({{ "/assets/media/2019-01-24/routebrowserparameter.PNG" | absolute_url }})

If we do not specify a route parameter, the user is returned a 404 as no matching route is found.

![Route Parameter 404]({{ "/assets/media/2019-01-24/routenotfound.PNG" | absolute_url }})

## Making route parameters optional

Supplying function parameters using routing as demonstrated in my previous example is a great way of making the URL of our function more human readable and easier to consume. However, we have refactored our code to remove the friendly message to alert the user they have forgotten to supply the name parameter.

In order to add this message back into our function, we need to make our route parameter optional.
This can be achieved by adding a **?** to the parameter name in the route definition and the type in the function header parameters (if it is a value type in order to make it nullable).

Refactor the code to the following.

{% highlight csharp %}
[FunctionName("NameFunction")]
public static IActionResult Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = "GetName/{name?}")]
    HttpRequest req,
    string name,
    ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    return name != null
        ? (ActionResult)new OkObjectResult($"Hello, {name}")
        : new BadRequestObjectResult("Please pass a name as a route parameter");
}
{% endhighlight %}

Now by specifying our name as a URL parameter, we can see our greeting message as expected.

![Route Parameter]({{ "/assets/media/2019-01-24/routebrowserparameter.PNG" | absolute_url }})

And by dropping the name, the parameter defaults to null and our reminder message is returned.

![Route Parameter Not Found]({{ "/assets/media/2019-01-24/routebrowserparameterdefault.PNG" | absolute_url }})

## Wrap Up

Hopefully this serves as a fairly comprehensive guide on how to configure routing on Azure Functions. If you have any questions, please let me know by Twitter (@TomLingDev) or the comment section below.