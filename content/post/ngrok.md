---
title: "Dev tools: ngrok"
date: 2018-04-08T23:47:27+04:30
tags: [tools,ngrok]
draft: false
---
Have you ever wanted to show your work which is running on your local machine to your
colleagues or friends? If they are far away of you, how will you show them? Will you upload
your source code to github to share it with your friends? But what if you don't like to
share your source code with others? Will you run it on public or private cloud or something like that?
But there is an awesome tool to expose your local services to the internet. Meet [ngrok](https://ngrok.com/).

## What is ngrok?
[ngrok](https://ngrok.com/) is a reverse proxy that creates a secure tunnel from a public endpoint to a locally
running web service. ngrok captures all traffic over the tunnel for later inspection and replay.

## What is ngrok useful for?

* Temporarily sharing a website that is only running on your development machine
* Demoing an app at a hackathon without deploying
* Developing any services which consume webhooks (HTTP callbacks) by allowing you to replay those requests
* Debugging and understanding any web service by inspecting the HTTP traffic
* Running networked services on machines that are firewalled off from the internet

## Demo
I created a demo application to show how it look like. This is my simple web application on port 8080 on my computer:
{{< fluid_imgs
        "center|/static/ngrok/d1.png|localhost"
>}}
Now you need [ngrok](https://ngrok.com/). After you downloaded ngrok and after you set your authentication 
token you can easily expose your local service to the Internet by typing:
```
./ngrok http 8080
```
It gives you a public url. In this case my given public url is **cdd65d2b.ngrok.com**  
{{< fluid_imgs
        "center|/static/ngrok/d2.png|ngrok"
>}}
Ngrok comes with a built-in dashbord (default on port 4040) that you can see all requests and replay a request:
{{< fluid_imgs
        "center|/static/ngrok/d3.png|dashboard"
>}}

Sources:

* [https://github.com/inconshreveable/ngrok](https://github.com/inconshreveable/ngrok)
* https://ngrok.com/


