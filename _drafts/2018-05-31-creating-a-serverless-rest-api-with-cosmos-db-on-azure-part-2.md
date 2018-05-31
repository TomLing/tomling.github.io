---
layout: post
title: Creating a serverless REST API with Cosmos DB on Azure (Part 2) - Create and Update operations
categories: [azure,csharp,dotnet]
tags: [csharp,cosmosdb,serverless,rest,api,azure,dotnet]
description: In this post we expand our function application to add the ability to update and create records in our database.
comments: true
---

## Expanding our serverless REST API

If you haven't yet - It would be good idea to get started by reading my [previous blog post]({% post_url 2018-02-18-creating-a-serverless-rest-api-with-cosmos-db-on-azure %}) detailing how to provision the Azure Function and Cosmos DB, Adding data to the database & creating a basic HTTP trigger function that returns data.

In this article we will expand our function that we created in our previous application by adding a function to add and update records in our Cosmos DB instance. We will also migrate our code from the IDE within the Azure Portal and use the Function SDK within Visual Studio. Using Visual Studio is hugely beneficial because it gives access to a number of tools to increase productivity and source control integration. 

## Installing the Azure Functions SDK

First off you will need to install Visual Studio (if you haven't already) and get the SDK. This can be achieved by navigating to Tools → Extensions and Updates. On the left pane click "Online" and search for and install "Azure Functions and Web Job Tools". This extension will install all of the required tools you will need to build and deploy Azure Functions in Visual Studio.

## Creating the function project

After installing the Azure function SDK we can now create a function project. You can do this by navigating to File → New Project → Search for "Azure Functions" and select the first result → Give the project a name and location → Click OK to confirm.

![New Post Options]({{ "/assets/media/2018-05-31/newpostoptions.PNG" | absolute_url }})

The next prompt confirms the trigger settings for the function - In this example we are going to choose the storage account that was created in my previous article (or use the emulator if you do not have an Azure Account). For the trigger itself we will create an anonymous HTTP trigger. Make sure that Azure Functions v1 is chosen from the top dropdown - At the time of writing Azure Functions v2 for .NET Standard is in preview.

![New Function Options]({{ "/assets/media/2018-05-31/newfunctionoptions.PNG" | absolute_url }})

Once our function has been created we will first need to import the required nuget packages, these include Newtonsoft - A library for working with JSON in .NET and the extension for Cosmos DB interfacing. In order to do this we will need to edit the .csproj file inside our solution. This can be achieved by right clicking the project inside solution explorer and clicking "Edit PROJECTNAME.csproj". 

Once the project file has been opened for editing amend the package references to look like the following -

{% highlight xml %}
<ItemGroup>
    <PackageReference Include="Microsoft.Azure.WebJobs.Extensions.DocumentDB" Version="1.2.0" />
    <PackageReference Include="Microsoft.NET.Sdk.Functions" Version="1.0.13" />
    <PackageReference Include="Newtonsoft.Json" Version="11.0.2" />
    <PackageReference Include="Newtonsoft.Json.Schema" Version="3.0.10" />
</ItemGroup>
{% endhighlight %}

Try rebuilding the solution to make sure that the nuget packages restore correctly without error. You may notice that Visual Studio gives a warning around version constraints with Newtonsoft.Json, curiously within the Microsoft.NET.Sdk.Functions nuget project version 9.0.1 of Newtonsoft.Json is marked as being required. Doing a bit of reading online, it has been discussed that this is due to runtime errors in certain circumstances; I however have not found any issues relating to using version 11.0.2 so I think it is safe to ignore this warning. If you do however encounter any problems let me know!

Now that we have imported the nugets we require we can now move onto creating our first Azure Function class.  



















