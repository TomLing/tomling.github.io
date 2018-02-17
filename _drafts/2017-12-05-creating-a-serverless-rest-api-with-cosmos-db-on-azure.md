---
layout: post
title: Creating a serverless REST API with Cosmos DB on Azure
categories: [azure,csharp,dotnet]
tags: [csharp,cosmosdb,severless,rest,api,azure,dotnet]
description: In this post I demonstrate how to create a serverless REST API using Cosmos DB easily on the Azure platform.
comments: true
---

## What is severless computing?

You've probably heard of the term serverless computing (or serverless functions) by now – but if you haven’t - in a nutshell serverless computing is all about abstracting the user away from the complexity of provisioning, maintaining and configuring servers and letting the developer focus on what they do best - coding. 

Microsoft Azure offer their own serverless implementation that is called Functions. Functions are simply pieces of code that are hosted in the cloud that can be triggered by numerous types of events. These can take the form of timers, data manipulation events and even HTTP calls. In the following example we will cover creating a simple REST API with an Azure Cosmos DB as the data store. Cosmos DB (formally known as Document DB) is Azure’s implementation of a No SQL database. 

At the moment is it free to get up and running with Azure – When you create a new account you are given £150 ($200) of credit for a month to use to experiment with most of the services that Azure has to offer. 


## Creating the services in Azure

First, we are going to create the Cosmos DB instance. Navigate to the new resource menu -> Search for Azure Cosmos DB and fill out the following account settings. Here are the settings I used – In this example I choose to use the SQL API for querying data but feel free to chose an option you are familiar with.

Now we need to add a collection of data to our database