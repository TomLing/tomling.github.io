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


## Creating the Cosmos DB service in Azure

First, we are going to create the Cosmos DB instance. Navigate to the new resource menu → Search for Azure Cosmos DB and fill out the following account settings. Here are the settings I used-

![Cosmos DB settings]({{ "/assets/media/cosmosdbsettings.png" | absolute_url}})

*Cosmos DB creation settings*

In this example I choose to use the SQL API for querying data but feel free to choose an option you are familiar with. Now we need to add a collection of data to our database to see our REST API working. For the purpose of this demonstration I have created a small data set of blog posts in JSON format which you can download [here.]({{ "/download/posts.json" | absolute_url }})

## Adding a data collection to our Cosmos DB instance

Next we need to download the Azure Cosmos DB database migration tool which can be found at the following link - https://docs.microsoft.com/en-us/azure/cosmos-db/import-data.
Once the tool is opened click next on the welcome screen → On the source information screen leave the 'Import from' drop down as 'JSON files' then click next. 

On the target information screen we need to enter the connection string for our Cosmos DB instance and specify a collection name for our JSON data. The tool demands the connection string in the following format-

{% highlight plaintext %}
AccountEndpoint={CosmosDB Endpoint};AccountKey={CosmosDB Key};Database={CosmosDB Database};
{% endhighlight %}

The 'AccountEndpoint' and 'AccountKey' parameters can be obtained from Azure by navigating to the 'Settings' section of our Cosmos DB instance and clicking the 'Keys' option.

![Cosmos DB keys]({{ "/assets/media/cosmosdbconnectionstrings.PNG" | absolute_url }})

*Cosmos DB key settings (keys have been redacted)*

So to format the connection string for the data migration tool to use, simply copy and paste the 'Primary Connection String' text from Azure and append "Database={CosmosDB Database};" by replacing '{CosmosDB Database}' with the name of the Cosmos DB instance that you chose. You can click 'Verify' to confirm you got everything correct.

For the collection field I am going to use 'posts' to reflect the type of data we are working with.

The partition key field can be left blank and the collection throughput can be set to the lowest value of 400.

The 'Id Field' section requires the Id field on our JSON data collection (think of this as the primary key). In my example it is simply 'id'.
Now we can simply click through all the windows and click the import button. The JSON data should now be imported into our Cosmos DB.

![Data migration tool import]({{ "/assets/media/dmtimport.PNG" | absolute_url }})

*Create the Azure Function*







