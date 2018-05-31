---
layout: post
title: Creating a serverless REST API with Cosmos DB on Azure (Part 1)
categories: [azure,csharp,dotnet]
tags: [csharp,cosmosdb,serverless,rest,api,azure,dotnet]
description: In this post I demonstrate how to create a serverless REST API using Cosmos DB easily on the Azure platform.
comments: true
---

## What is serverless computing?

You've probably heard of the term serverless computing (or serverless functions) by now – but if you haven’t - in a nutshell serverless computing is all about abstracting the user away from the complexity of provisioning, maintaining and configuring servers and letting the developer focus on what they do best - coding. 

Microsoft Azure offer their own serverless implementation that is called Functions. Functions are simply pieces of code that are hosted in the cloud that can be triggered by numerous types of events. These can take the form of timers, data manipulation events and even HTTP calls. In the following example we will cover creating a simple REST API with an Azure Cosmos DB as the data store. Cosmos DB (formerly known as Document DB) is Azure’s implementation of a No SQL database. 

At the moment is it free to get up and running with Azure – When you create a new account you are given £150 ($200) of credit for a month to use to experiment with most of the services that Azure has to offer. 


## Creating the Cosmos DB service in Azure

First, we are going to create the Cosmos DB instance. Navigate to the new resource menu → Search for Azure Cosmos DB and fill out the required creation settings. Here are the settings I used -

![Cosmos DB settings]({{ "/assets/media/2018-02-18/cosmosdbsettings.png" | absolute_url}})

*Cosmos DB creation settings*

In this example I choose to use the SQL API for querying data but feel free to choose an option you are familiar with. Now we need to add a collection of data to our database to see our REST API working. For the purpose of this demonstration I have created a small data set of blog posts in JSON format which you can download [here.]({{ "/download/posts.json" | absolute_url }})

## Adding a data collection to our Cosmos DB instance

Next, we need to download the Azure Cosmos DB database migration tool which can be found at the following link - https://docs.microsoft.com/en-us/azure/cosmos-db/import-data.
Once the tool is opened click next on the welcome screen → On the source information screen leave the 'Import from' drop down as 'JSON files' then click next. 

On the target information screen, we need to enter the connection string for our Cosmos DB instance and specify a collection name for our JSON data. The tool demands the connection string in the following format -

{% highlight plaintext %}
AccountEndpoint={CosmosDB Endpoint};AccountKey={CosmosDB Key};Database={CosmosDB Database};
{% endhighlight %}

The 'AccountEndpoint' and 'AccountKey' parameters can be obtained from Azure by navigating to the 'Settings' section of our Cosmos DB instance and clicking the 'Keys' option.

![Cosmos DB keys]({{ "/assets/media/2018-02-18/cosmosdbconnectionstrings.PNG" | absolute_url }})

*Cosmos DB key settings (keys have been redacted)*

So to format the connection string for the data migration tool to use, simply copy and paste the 'Primary Connection String' text from Azure and append "Database={CosmosDB Database};" by replacing '{CosmosDB Database}' with the name of the Cosmos DB instance that you chose. You can click 'Verify' to confirm you got everything correct.

For the collection field I am going to use 'posts' to reflect the type of data we are working with.

The partition key field can be left blank and the collection throughput can be set to the lowest value of 400 (cheapest cost).

The 'Id Field' section requires the Id field on our JSON data collection (think of this as the primary key). In my example it is simply 'id'.
Now we can simply click through all the windows and click the import button. The JSON data should now be imported into our Cosmos DB.

![Data migration tool import]({{ "/assets/media/2018-02-18/dmtimport.PNG" | absolute_url }})

*Data migration tool import*

## Creating the Azure Function App

Finally, we are going to create the Azure Function that will act as our REST API. Navigate back to the new resource menu → Search for Function App and fill out the required creation settings. I recommend using the consumption plan for hosting due to having 1 million free executions per month.
Here are the settings I used -

![Function App settings]({{ "/assets/media/2018-02-18/functionsettings.PNG" | absolute_url }})

*Function App creation settings*

Now we have created the Function App we need to create the actual REST API function. To do this go to the Function App resource → Click the '+' button next to the 'Functions' option, on the next pane, click the 'Custom function' option.

You will now be presented with a number of template options - Choose the C# HTTP trigger → Give the new function a name and authorisation level (In this example we will use Anonymous), then click create.

After the function has been created a pane should display with the C# code that forms our HTTP API function. In order to pass in the data from our Cosmos DB as a parameter to the function we need to add a new 'Azure Cosmos DB input'. This can be achieved by expanding our function in the left pane and clicking the 'Integrate' option.

![Function Integrate Option]({{ "/assets/media/2018-02-18/functionintegrate.PNG" | absolute_url }})

On the new window that has been presented click the 'New Input' option at the top of the screen, select Azure Cosmos DB and match the values in the following screenshot. To add a new account connection, click the 'new' button and remember that the 'Database name' field is the name of the Azure Cosmos DB instance we created earlier.

![Function Input Options]({{ "/assets/media/2018-02-18/functioninput.PNG" | absolute_url }})

Before hitting the 'Save' button take note of the SQL Query that will filter out the data that will be available to our function - In our case it will return all records in our 'posts' collection. (The 'c' in this context is an alias for our posts collection). Now hit the save button!

Now it's time to edit the function code in order to return all records in our posts collection. I order to do this click the name of the function in the left pane and amend the C# code presented to the following -

{% highlight csharp %}
using System.Net;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log, IEnumerable<dynamic> posts)
{
    return req.CreateResponse(HttpStatusCode.OK, posts);
}

 
{% endhighlight %}

Basically, what this code is doing is passing in our input we just configured as an collection of dynamic objects. The dynamic type is used because we do not know the schema of our post object until the function is executed at runtime. Note that the name of the parameter matches up with how we configured our input previously. 

Next, we can actually execute our function by clicking the 'Save and run' button, and relying on that your got everything configured correctly, the list of posts in JSON format should appear in the 'Output' window.

## Wrap Up

Hopefully this has given you a greater insight into the power of setting up an Azure Function for tasks such as returning data and performing operations in the cloud without having to provision or maintain server hardware. This example obviously demonstrates a fairly basic operation but there is no end of use cases when using Functions on Azure.

In the next article I will show how to add other data manipulation options to our function such as adding and updating records in the Azure Cosmos DB.














