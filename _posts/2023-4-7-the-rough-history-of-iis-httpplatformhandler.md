---
layout: post
title: "The Rough History of IIS HttpPlatformHandler"
description: "This post talks about the rough history of IIS HttpPlatformHandler and the miseries around it."
tags: .NET Visual-Studio Windows IIS
excerpt_separator: <!--more-->
---

Due to various misinformation around this IIS out-of-band component, I think it's worth the while to write about its history so you know what others are talking about and how some of them made mistakes.
<!--more-->

## The Start of Windows Azure

When Microsoft entered the cloud computing market, it was a big challenge to support non-Microsoft technologies on Windows Server so that this platform can expand its reach to a broader audience.

That effort happened to align with a few important changes on IIS itself, where FastCGI support was added so that PHP applications can be hosted on IIS via that interface. PHP community was willing to help on PHP for Windows while Microsoft provided support.

While this was a big step forward, soon people discovered the difficulty to extend the excitement to other programming languages, such as Ruby and Python. While it is possible to develop the FastCGI based integration for Python, which later became the open source wfastcgi module, who was going to maintain the FastCGI integration bits? As a result, the Python wfastcgi module had been maintained by Microsoft developers till a while ago. But this didn't work out for Ruby, Go, Java, Node.js and many more.

## The Reverse Proxy Disadvantages

Clearly an alternative is to run those non Microsoft technologies on their own application servers, and then IIS/ARR can sit in front as reverse proxy. The popularity of nginx proved that approach for long, but it comes with its own disadvantages,

1. The application server is usually running on its own, so you should not expect it to work in your familiar application pool pattern that from time to time the processes are recycled.
1. You need complex reverse proxy rules to be defined on IIS side, so that traffic can be properly forwarded to the application server.
1. You need to predetermine the port for your application server. It's not a big deal for your own server which just hosts one or two web apps this way, but what about an enterprise server with hundreds of web apps? It's not easy to manage the port numbers.

## The Magical HttpPlatformHander

The experience of building FastCGI and ARR gave Microsoft developers enough experience on both sides of the coin, so they came back with a very smart idea and implemented a new component called HttpPlatformHandler.

In general it works as a reverse proxy between IIS and your application server, but it hooks up the reverse proxy rules itself and also determines which port to use. Then all you need is to specify which process should be called with what arguments to launch the application server process. After such simple configuration, this module spins out the application server on demand and recycle it whenever needed. The port number to use is either passed as one of the arguments or via an environment variable.

The simplicity of this design makes the module a great success, as you no longer need any specific FastCGI bits or ISAPI modules. All you need is just the very simple HttpPlatformHandler configuration snippet and it works for almost all modern web stacks.

## The Mist of ASP.NET Core

Naturally when Microsoft designed ASP.NET Core, HttpPlatformHandler was chosen to be the integration point. However, the more features ported from ASP.NET 4.x, the more gaps identified that HttpPlatformHandler cannot fulfill all the needs.

ASP.NET 4.x has so tight an integration mode with IIS (if you read about the integrated pipeline), that a dedicated IIS module must be developed upon HttpPlatformHandler to support most of the features and enable a smooth migration path. Of course, this new module is now known as ASP.NET Core module for IIS.

Almost all misinformation today can be traced back to [this announcement made by the ASP.NET Core team on GitHub](https://github.com/aspnet/IISIntegration/issues/105). Readers without knowing the entire background or reading the announcement carefully can easily assume that they shouldn't use HttpPlatformHandler, but switch to ASP.NET Core module. But you can see how wrong that assumption is.

## Conclusion

HttpPlatformHandler is still fully supported by Microsoft, and has been upgraded to support all latest Windows releases. If you are hosting Python/Ruby/Go/Java/Node.js applications on IIS, give it a try and feel how simple it is to set up everything around it.

## Side Notes
HttpPlatformHandler is also enabled on Azure App Service (Windows) by default, so if you have tested your web apps on your own IIS server, you can confidently host them on Azure App Service without many changes on configuration.

I also created the necessary PowerShell scripts to help you enable HttpPlatformHandler on IIS Express if you want to give it a try. You can find them on [my GitHub repository](https://github.com/lextm/iisexpress-httpplatformhandler).

## References

* [Download](https://www.iis.net/downloads/microsoft/httpplatformhandler#additionalDownloads)
* [Configuration Reference by Microsoft](https://learn.microsoft.com/iis/extensions/httpplatformhandler/httpplatformhandler-configuration-reference)
* [Java Example by Microsoft](https://learn.microsoft.com/previous-versions/azure/windows-server-azure-pack/mt125371(v=technet.10))
* [Python Example by Microsoft](https://learn.microsoft.com/visualstudio/python/configure-web-apps-for-iis-windows?view=vs-2022#configure-the-httpplatform-handler)
* [Python Example by me](/running-flask-web-apps-on-iis-with-httpplatformhandler/)
* [Node.js Example by me](https://halfblood.pro/running-nodejs-web-apps-on-iis-with-httpplatformhandler/)
* [Ruby Example by Scott Hanselman](http://www.hanselman.com/blog/announcing-running-ruby-on-rails-on-iis8-or-anything-else-really-with-the-new-httpplatformhandler)
* [Go Example by Hang Ruan](https://blog.jayway.com/2015/02/16/deploying-go-applications-azure-websites/)