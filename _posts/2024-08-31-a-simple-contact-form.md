---
layout: post
title: 'A stupidly simple contact form'
date: 2024-08-31 22:00 -0400
description: 'A simple way to add a working contact form to your website using Discord webhooks — no mail services or complex setups needed'
image: /assets/img/contact_form_screenshot.png
categories: ['Quick Tips']
---

This is a quick little post to tell you that platforms like discord offer webhooks to send messages. **USE THEM!**

## Background
For some context, I was wanting to add a contact section to [britto.tech](https://britto.tech) but was hesitant. 
This is because I wanted to keep the site as simple as possible (I only use vanilla HTML & CSS along with a tiny amount of JS for the website)
and didn't feel like going through the hassle of setting up an email sending service. 
I also hated the alternative of opening an email client like a lot of personal websites do. 
Also if someone wanted to contact me, my Email and LinkedIn are right there on the page. 
They can use it. Due to this I did not really give it much thought and moved on to other things.

Last week I had a thought. Discord offers webhooks to send messages. 
I have used this before to send events from github to a channel on our discord server a while ago when I worked on a project with some of friends. 
Why don't I use that to send messages from my website? That was it. 
I did not know how or why that thought came to my mind or why haven't I thought of this earlier. 
I haven't heard of this being mentioned anywhere before. 
Maybe I am too out of touch with the tech space to hear about this. 
Maybe a large number of forms that actually work instead of opening an email client might be using something like this. 
Maybe I am dumb and this was too obvious of a solution that no one felt like mentioning it. 
Anyway, doesn't matter, I finally have a simple way to send messages from my website!

## A super quick tutorial
This is super easy. If you read discord's [docs](https://discord.com/developers/docs/resources/webhook) you can see it's just one POST request and you're all set. 
No auth. No hassles. 
Below is a simple fetch request that sends a message on discord:
```js
    fetch('WEBHOOK_URL_HERE', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({
            content: 'Hello World! Your content goes here'
        }),
    }).then(response => {
        if (!response.ok) {
            // Show Error
        }
        else {
           // Success!
        }
    }).catch(error => {
            // Error :(
    });
```
This is as basic as it gets. 
It is just a POST request with the message you want to send. 
There are more attributes that you can add instead of/along with the content attribute 
like embeds or username to make it prettier to view in discord. 
There are some limits as well that are enforced by discord to prevent abuse. 
Read the [docs](https://discord.com/developers/docs/resources/webhook) and you'll understand.

To get the webhook url you have to create a webhook in a discord channel. Preferably on a server you are the owner of.  
Go to the channel you want messages to be sent in, click the settings icon near on the side bar right next to the channel name, 
under integrations, create a new webhook and copy its url. 
Now you can paste it in the fetch call and execute.

Tada 🎉 You sent yourself a message!

<video autoplay loop muted style="border-radius: 10px; width: 100%">
  <source src="/assets/vid/Screencast from 2024-08-31-webhook.webm" type="video/webm">
</video>

> The webhook url follows this format `https://discord.com/api/webhooks/WEBHOOK_ID/TOKEN`
{: .prompt-info }

## Security
In discord, webhooks are better isolated in contrast to their bot API. 
With bots if someone gets access to your token they can abuse whatever resources the bot has access to 
which probably is multiple servers with many users. 
However webhooks have no way of interacting with most things 
other than sending message in the channel it was created in and modify/delete its own messages. 

So that means someone can modify earlier messages sent by someone else? 

Well, not really. 

To modify or delete a message you need its id, which is not that easy to guess. 
A brute force attack for the id also won't help due to discord's rate-limiting.

Okay, then leaving your webhook url in javascript should be safe? Right?

Again, no. Ideally you should not.

There is one dangerous endpoint that discord allows to be used with just the webhook token and id. Deleting the webhook itself. This is just unfortunate. This means that someone can copy the token from your webhook url and do a DELETE request and BOOM! Your contact form is broken. 😥

This is bad. 
I can no longer use just a static html page with no backend server for my website if I want to have a contact form. 
Maybe I should look into something like slack or telegram. 
Maybe they have webhooks. 
Those weren't an option as I don't use them often and therefore might miss the messages being sent once in a blue moon.

## A solution?
The solution to this problem is pretty simple if you already have a vps used for something else or a free tier one. 
We need to store the token on the vps and send messages through it to discord. 
I used C# to create a simple API to proxy the messages. 
It took only a few lines of code. I have pushed it to [github](https://github.com/JoseBritto/DiscordMessageHook) if anyone wants to take a look. 
Here's the Program.cs file in its  entirety:

```cs
using System.Text;
using Microsoft.AspNetCore.Mvc;
 
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", x =>
    {
           x.AllowAnyOrigin()
            .AllowAnyHeader()
            .AllowAnyMethod();
    });
});
var app = builder.Build();
app.UseCors("AllowAll");
app.UseHttpsRedirection();
 
app.MapPost("/sendMessage", async (HttpContext context, string webhookId) =>
{
    var token = app.Configuration[$"Tokens:{webhookId}"];
    if(token == null)
    {
        app.Logger.LogInformation("Unauthorized request received for {webhookId}", webhookId);
        return Results.Unauthorized();
    }
    using var reader = new StreamReader(context.Request.Body, Encoding.UTF8);
    var json = await reader.ReadToEndAsync();
    app.Logger.LogInformation("Received Message from {webhookId}", webhookId);
    app.Logger.LogInformation("Forwarding content: {json}", json);
    using var httpClient = new HttpClient();
    var uri = new Uri($"https://discord.com/api/webhooks/{webhookId}/{token}?wait=true");
    var result = await httpClient.PostAsync(uri, new StringContent(json, Encoding.UTF8, "application/json"));
    var responseBody = await result.Content.ReadAsStringAsync();
    Console.WriteLine(responseBody);
    return Results.Content(responseBody, result.Content.Headers.ContentType?.MediaType, statusCode: (int)result.StatusCode);
});
var port = app.Configuration["Port"] ?? "4500";
var ip = app.Configuration["IP"] ?? "127.0.0.1";
app.Urls.Add($"http://{ip}:{port}");
app.Run();
```

Now all I have to do is to replace the webhook url in javascript with 
`my.domain?webhookId=Id` where `my.domain` is where I hosted this service and 
`Id` is the id of the webhook. 
Given that I have the webhook id and token setup in the `appsettings.json` file on the server, 
it should all work exactly the same as before but now with more security. 
Since you have more control here, you can even try to tackle spam messages if you have that problem.

