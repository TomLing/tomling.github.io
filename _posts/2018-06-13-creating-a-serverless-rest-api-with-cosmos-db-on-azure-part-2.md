---
layout: post
title: Creating a serverless REST API with Cosmos DB on Azure (Part 2) - Create and Update operations
categories: [azure,csharp,dotnet,serverless,functions]
tags: [csharp,cosmosdb,serverless,rest,api,azure,dotnet,functions]
description: In this post we expand our function application to add the ability to update and create records in our database.
comments: true
---

## Expanding our serverless REST API

If you haven't yet - It would be good idea to get started by reading my [previous blog post]({% post_url 2018-02-18-creating-a-serverless-rest-api-with-cosmos-db-on-azure %}) detailing how to provision the Azure Function and Cosmos DB, Adding data to the database & creating a basic HTTP trigger function that returns data from a Cosmos DB collection.

In this article we will expand our function that we created in our previous application by adding a function to add and update records in our Cosmos DB instance. We will also migrate our code from the IDE within the Azure Portal and use the Function SDK within Visual Studio. Using Visual Studio is hugely beneficial because it gives access to a number of tools to increase productivity and source control integration.

## Installing the Azure Functions SDK

First off you will need to install Visual Studio (if you haven't already) and get the SDK. This can be achieved by navigating to Tools → Extensions and Updates. On the left pane click "Online" and search for and install "Azure Functions and Web Job Tools". This extension will install all of the required tools you will need to build and deploy Azure Functions in Visual Studio.

## Creating the function solution

After installing the Azure function SDK we can now create a function project. You can do this by navigating to File → New Project → Search for "Azure Functions" and select the first result → Give the project a name and location → Click OK to confirm.

![New Post Options]({{ "/assets/media/2018-06-13/newpostoptions.PNG" | absolute_url }})

The next prompt confirms the trigger settings for the function. In this example we are going to choose the storage account that was created in my previous article (or use the emulator if you do not have an Azure Account). For the trigger itself we will create an anonymous HTTP trigger. Make sure that Azure Functions v1 is chosen from the top dropdown - At the time of writing Azure Functions v2 for .NET Standard is in preview.

![New Function Options]({{ "/assets/media/2018-06-13/newfunctionoptions.PNG" | absolute_url }})

Once our function has been created we will first need to import the required nuget packages, these include Newtonsoft - A library for working with JSON in .NET and the extension for Cosmos DB interfacing. In order to do this, we will need to edit the .csproj file inside our solution. This can be achieved by right clicking the project inside solution explorer and clicking "Edit YOUR_PROJECT_NAME.csproj". 

Once the project file has been opened for editing amend the package references to look like the following

{% highlight xml %}
<ItemGroup>
    <PackageReference Include="Microsoft.Azure.WebJobs.Extensions.DocumentDB" Version="1.2.0" />
    <PackageReference Include="Microsoft.NET.Sdk.Functions" Version="1.0.13" />
    <PackageReference Include="Newtonsoft.Json" Version="11.0.2" />
    <PackageReference Include="Newtonsoft.Json.Schema" Version="3.0.10" />
</ItemGroup>
{% endhighlight %}

Try rebuilding the solution to make sure that the nuget packages restore correctly without error. You may notice that Visual Studio gives a warning around version constraints with Newtonsoft.Json, curiously within the Microsoft.NET.Sdk.Functions nuget project version 9.0.1 of Newtonsoft.Json is marked as being required. Doing a bit of reading online, it has been sed on various issue threads in Github that this is due to runtime errors in certain circumstances; I however have not found any issues relating to using version 11.0.2 so I think it is safe to ignore this warning. If you do however encounter any problems let me know!

## Creating the functions

Now that we have imported the nugets we require we can now move onto creating our first Azure Function class. The first piece of functionality we are going to create is a simple GET method that will return all records in our Cosmos DB posts collection (similar to what we created in my previous post). The first thing we need to do is rename the existing class file that was created we our function solution, in this example I will name it "GetPosts". Here is the code for the function (Modify the code to enter details of your own Cosmos DB instance.)

{% highlight csharp %}
public static class GetPosts
    {
        [FunctionName("GetPosts")]
        public static HttpResponseMessage Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = null)]
            HttpRequestMessage req,
            TraceWriter log,
            [DocumentDB("YOUR_COSMOS_DB_INSTANCE_NAME", "posts",
            ConnectionStringSetting = "CosmosDbConnection")]
            IEnumerable<object> posts)
        {
            return req.CreateResponse(HttpStatusCode.OK, posts);
        }
    }
{% endhighlight %}

There are a couple of things to note in this code that you may not be familiar with if you have only previously used the Azure IDE to create a function. Firstly, that input parameters have attributes that determine settings such as authorization levels, routing and database connection strings. Notice that one of our input parameters is a connection of posts that represent all of the items in our Cosmos DB collection. In order to connect to our database we need to specify the Cosmos DB connection string provided in the Azure portal and copy this into the local.settings.json within our project. 

Your local.settings.json file should look similar to this. Your Cosmos DB instance connection string can be found under "Keys" in the Azure portal

{% highlight json %}
{
    "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "YOUR_STORAGE_ACCOUNT_CONNECTION_STRING",
    "AzureWebJobsDashboard": "YOUR_STORAGE_ACCOUNT_CONNECTION_STRING",
    "CosmosDbConnection": "YOUR_COSMOS_DB_CONNECTION_STRING"
  }
}
{% endhighlight %}

With all this in place we are now ready to give our function a test. Run the function by pressing F5. The following console window should display detailing the endpoint URL for our function.

![Function Debugging]({{ "/assets/media/2018-06-13/functionconsole.PNG" | absolute_url }})

In order to send a request to our API to retrieve data back we will use a tool called Postman. First download the application, select GET as the method call, enter the URL in the console window and press the send button.

If you have any records in your posts collection you should see them returned in the results windows, along with a status code of 200 (OK).

![Postman results]({{ "/assets/media/2018-06-13/postmanresults.PNG" | absolute_url }})

The second function we are going to create has the functionality to create and update posts in our database. As with our previous function create a new class file by right clicking our project file and clicking Add → New Item, search for Azure Function and name it "CreateOrUpdatePost". Again, we will create an anonymous Http trigger.

Here is the code for our create or update function

{% highlight csharp %}
public static class CreateOrUpdatePost
    {
        [FunctionName("CreateOrUpdatePost")]
        public static HttpResponseMessage Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "put", "post", Route = null)] 
            HttpRequestMessage req,
            TraceWriter log,
            [DocumentDB("YOUR_COSMOS_DB_INSTANCE_NAME", "posts", Id = "id",
            ConnectionStringSetting = "CosmosDbConnection")] out object post)
        {
            try
            {
                var requestBody = req.Content.ReadAsStringAsync();

                var generator = new JSchemaGenerator();
                var schema = generator.Generate(typeof(Post));
                var rawPost = requestBody.Result;
                var isPost = JObject.Parse(rawPost).IsValidJson(schema);

                if (isPost)
                {
                    var deserializePost = JsonConvert.DeserializeObject<Post>(rawPost);
                    post = new {
                            userId = deserializePost.UserId.ToString(),
                            id = deserializePost.Id.ToString(),
                            title = deserializePost.Title,
                            body = deserializePost.Body };
                    return req.CreateResponse(HttpStatusCode.OK, post);
                }
                post = null;
                return req.CreateResponse(HttpStatusCode.BadRequest,
                    "Please pass a post on the query string or in the request body");
            }
            catch (Exception ex)
            {
                post = null;
                return req.CreateResponse(HttpStatusCode.InternalServerError,
                    $"The following error occurred: {ex.Message}");
            }
        }
    }

    public static class ValidateJson
    {
        public static bool IsValidJson(this JObject jObject, JSchema jSchema)
        {
            return jObject.IsValid(jSchema);
        }
    }

    public class Post
    {
        [JsonProperty(PropertyName = "userId")]
        public int UserId { get; set; }
        [JsonProperty(PropertyName = "id")]
        public int Id { get; set; }
        [JsonProperty(PropertyName = "title")]
        public string Title { get; set; }
        [JsonProperty(PropertyName = "body")]
        public string Body { get; set; }
    }
{% endhighlight %}

This code works by taking a post in JSON format, then it will validate the raw JSON against our post schema, if it matches successfully will add or update the record depending if there is a post already in the collection with a matching Id. Again we can test this function by using Postman. Start by debugging the application and copying the endpoint address.

![Function Debugging]({{ "/assets/media/2018-06-13/functionconsole2.PNG" | absolute_url }})

Open Postman and chose POST or PUT as the HTTP method, then paste the endpoint address into the URL section. Next click the "Body" tab, select the "raw" radio button and paste the following post JSON into the text section.

{% highlight json %}
{
    "userId" : 8,
    "id" : 123,
    "title" : "hello",
    "body" : "world"
}
{% endhighlight %}

Then hit the "Send" button. You should hopefully get a response including a status code of 200 (OK).

Next re-run our "GetPosts" method and depending on if you already had a post with id 123 in the database, you should see our post in our Cosmos DB collection.

![New Post Result]({{ "/assets/media/2018-06-13/newpostresult.PNG" | absolute_url }})

## Wrap Up

That concludes my two-part series on creating a basic serverless HTTP API using Cosmos DB in Azure. What I have shown here only really scratches the surface on what is possible using serverless functions in Azure. The possibilities are truly endless! If you have any questions please contact me using the comment section of this post and I will be happy to help.