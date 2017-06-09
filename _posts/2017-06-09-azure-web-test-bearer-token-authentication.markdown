---
layout: post
title:  "Azure Webtest and JWT bearer token authentication"
categories: azure-webtest, jwt, bearer-token
date:   2017-06-09 23:03:26 -0700
---
Hello, if you guys are trying to use Azure Webtest to maintain the availability of your web apps and running into the problem that WebTest needs to test against a secured web apis and still haven't figured it out yet then this is the article that you are looking for. Firstly, let's read these 2 for basic understanding [this article][page-1] from MS and [this article][page-2]. Ok, you read it right? If you got your solution then it's awesome, if you haven't? don't worry. Let's begin. What you need to do is preparing:

* An application in AAD, go to Configure section of your app and you will have (**CLIENT ID**, **CLIENT SECRET (KEY)**)
* Go to Visual Studio and create new WebTest project. Let's set up everything like this. 
![Screenshot]({{ site.url }}/assets/687474703a2f2f692e696d6775722e636f6d2f6a6934303233302e706e67.PNG)

**Block 1**: your tenant id or tenant uri

**Block 2**: important it can be either your client ID or APP ID URI (they are all in Azure AD configure page). I will talk about it later.

**Block 3**: client id

**Block 4**: client secret

**Block 5**: your secured api to be tested.

Ok, so let's talk about block 2. This value really depends on how your server sets up. Here, I will use .net core as an example. Here is how I set up my app.
![Screenshot]({{ site.url }}/assets/687474703a2f2f692e696d6775722e636f6d2f4f7163346270352e706e67.PNG)

My server expects client to send bearer token if user is authenticated. **Configuration["AzureAd:Audience"]** is my **client Id**. Here is the tricky part: The value of **Block 2** needs to be matched with **Audience value** inside **UseJwtBearerAuthentication**. It means if I am using client Id here then my block 2 value should be client Id as well. That's the most tricky part. Now if you look at the first image, after the first request you will get **AuthorizationHeader** and it contains the access token. So the next step is extract it and send it with the second request. Unfortunately, you will need to modify the code a little bit. Let's generate code from our Webtest. **this.Context["AuthorizationHeader"]** is a JSON object. This code will extract access token.

{% highlight C# %}
JToken token = JObject.Parse(this.Context["AuthorizationHeader"].ToString());

WebTestRequest request2 = new WebTestRequest("your_api_url");

request2.Headers.Add(new WebTestRequestHeader("Authorization", ("Bearer " + token.SelectToken("access_token").ToString())));
{% endhighlight %}

Happy coding!

Hung Cao