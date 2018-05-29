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

The next 




















