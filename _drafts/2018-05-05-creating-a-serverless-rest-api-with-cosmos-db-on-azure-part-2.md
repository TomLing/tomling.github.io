---
layout: post
title: Creating a serverless REST API with Cosmos DB on Azure (Part 2) - Create and Update operations
categories: [azure,csharp,dotnet]
tags: [csharp,cosmosdb,serverless,rest,api,azure,dotnet]
description: In this post we expand our function application to add the ability to update and create records in our database.
comments: true
---

## Expanding our serverless REST API

If you haven't yet - It would be good idea to get started by reading my previous blog post detailing how to provision the Azure Function and Cosmos DB, Adding data to the database & creating a basic HTTP trigger function that returns data.

In this article we will expand our function that we created in our previous application by adding a function to add and update records in our Cosmos DB instance. We will also migrate our code from the IDE within the Azure Portal and use the Function SDK within Visual Studio (You can use VS Code if you prefer). Using Visual Studio is hugely beneficial because it gives access to a number of tools to increase productivity and source control integration. 


















