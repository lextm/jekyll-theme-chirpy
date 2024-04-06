---
layout: post
title: "The Rough History of IIS HttpPlatformHandler"
description: "This post talks about the rough history of IIS HttpPlatformHandler and the miseries around it."
tags: .NET Visual-Studio Windows IIS HttpPlatformHandler
categories: [History, IIS]
excerpt_separator: <!--more-->
---

Due to various misinformation around this IIS out-of-band component, I think it's worth the while to write about its history so you know what others are talking about and how some of them made mistakes.

<!--more-->

## The Start of Windows Azure

When Microsoft entered the cloud computing market, it was a big challenge to support non-Microsoft technologies on Windows Server so that this platform can expand its reach to a broader audience.

That effort happened to align with a few important changes on IIS itself, where FastCGI support was added so that PHP applications can be hosted on IIS via that interface in 2006. Zend Technologies Inc. and PHP community were willing to help on PHP for Windows while Microsoft provided support.

While this was a big step forward, soon people discovered the difficulty to extend the excitement to other programming languages. For example, wfastcgi was developed by Microsoft Python team in 2012 to enable FastCGI support for Python. iisnode was developed by Tomasz Janczuk initially in 2012 to enable FastCGI support for Node.js, and later handed over to Microsoft Azure team.

But the FastCGI approach isn't sustainable, and the story for Ruby, Go, Java, and many more was undone.

## The Reverse Proxy Disadvantages

Clearly an alternative is to run those non Microsoft technologies on their own application servers, and then IIS/ARR can sit in front as reverse proxy. The popularity of nginx proved that approach for long, but it comes with its own disadvantages,

1. The application server is usually running on its own, so you should not expect it to work in your familiar application pool pattern that from time to time the processes are recycled.
1. You need complex reverse proxy rules to be defined on IIS side, so that traffic can be properly forwarded to the application server.
1. You need to predetermine the port for your application server. It's not a big deal for your own server which just hosts one or two web apps this way, but what about an enterprise server with hundreds of web apps? It's not easy to manage the port numbers.

## The Magical HttpPlatformHandler

The experience of building FastCGI and ARR gave Microsoft developers enough experience on both sides of the coin, so they came back with a very smart idea and implemented a new component called HttpPlatformHandler.

In general it works as a reverse proxy between IIS and your application server, but it hooks up the reverse proxy rules itself and also determines which port to use. Then all you need is to specify which process should be called with what arguments to launch the application server process. After such simple configuration, this module spins out the application server on demand and recycle it whenever needed. The port number to use is either passed as one of the arguments or via an environment variable.

The simplicity of this design makes the module a great success, as you no longer need any specific FastCGI bits or ISAPI modules. All you need is just the very simple HttpPlatformHandler configuration snippet and it works for almost all modern web stacks.

Initially this was used by Microsoft Azure Websites to host Java web apps, and later released as a separate download for IIS in 2015.

## The Mist of ASP.NET Core Module

Naturally when Microsoft designed ASP.NET Core, HttpPlatformHandler was chosen to be the integration point. However, the more features ported from ASP.NET 4.x, the more gaps identified that HttpPlatformHandler cannot fulfill all the needs.

ASP.NET 4.x has so tight an integration mode with IIS (if you read about the integrated pipeline), that a dedicated IIS module must be developed upon HttpPlatformHandler to support most of the features and enable a smooth migration path. Of course, this new module is now known as ASP.NET Core module for IIS.

Almost all misinformation today can be traced back to [this announcement made by the ASP.NET Core team on GitHub](https://github.com/aspnet/IISIntegration/issues/105). Readers without knowing the entire background or reading the announcement carefully can easily assume that they shouldn't use HttpPlatformHandler, but switch to ASP.NET Core module. But you can see how wrong that assumption is. Author of the original announcement later clarified that the announcement was not meant to discourage HttpPlatformHandler usage, but to inform the community about the new module in [this comment](https://github.com/aspnet/IISIntegration/issues/1454#issuecomment-425472537),

> "HttpPlatformHandler was only replaced by ANCM for ASP.NET Core apps, it is still maintained for other uses. You're welcome to use ANCM for other applications but we don't document or support that scenario, you would need to reverse engineer it from here."

There are a few noticeable things we can observe from ASP.NET Core module source code though (to help you better understand its sibling HttpPlatformHandler),

1. The module keeps the functionality of HttpPlatformHandler, so that it can host Kestrel processes in out-of-process mode. But you won't find the documentation from Microsoft on the port number environment variable, as ASP.NET Core/Kestrel has that knowledge built in.
1. Except the way to shut down the application server process via Control+C, it also adds a new way to shut down the process via a special URL (`/iisintegration`). This is a feature that HttpPlatformHandler does not have.
1. The module adds a lot of features to support ASP.NET Core in-process mode, which is not available in HttpPlatformHandler. This is why the module is not a replacement for HttpPlatformHandler, and it has a larger memory footprint.
1. It changed some internals such as [the 8kb internal buffer](https://github.com/aspnet/IISIntegration/issues/7). Those changes were not backported to HttpPlatformHandler.

> If you are interested in learning more about ASP.NET Core module, you might review its code base on GitHub under `dotnet/aspnetcore` repository. Methods like [`UseIISIntegration`](https://learn.microsoft.com/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderiisextensions.useiisintegration) or`UseIIS` are also important.

## Conclusion

HttpPlatformHandler is still fully supported by Microsoft, and has been upgraded to support all latest Windows releases. If you are hosting Python/Ruby/Go/Java/Node.js applications on IIS, give it a try and feel how simple it is to set up everything around it.

## Side Notes

HttpPlatformHandler is also enabled on Azure App Service (Windows) by default, so if you have tested your web apps on your own IIS server, you can confidently host them on Azure App Service without many changes on configuration.

I also created the necessary PowerShell scripts to help you enable HttpPlatformHandler on IIS Express if you want to give it a try. You can find them on [my GitHub repository](https://github.com/lextm/iisexpress-httpplatformhandler).

## References

- [PHP on IIS announcement](https://news.microsoft.com/2006/10/31/microsoft-and-zend-technologies-announce-technical-collaboration-to-improve-interoperability-of-php-on-the-windows-server-platform/)
- [wfastCGI initial commit](https://github.com/microsoft/PTVS/commit/0b944a292442dcb7a5caaffb9e3cd7542bbf190f)
- [iisnode initial commit](https://github.com/tjanczuk/iisnode/commit/2ad22f2dbc5d9721a58c006c5fb7aef18ae6b430)
- [HttpPlatformHandler initial announcement](https://azure.microsoft.com/blog/announcing-the-release-of-the-httpplatformhandler-module-for-iis-8/)
- [Download](https://www.iis.net/downloads/microsoft/httpplatformhandler#additionalDownloads)
- [Configuration Reference by Microsoft](https://learn.microsoft.com/iis/extensions/httpplatformhandler/httpplatformhandler-configuration-reference)
- [Java Example by Microsoft](<https://learn.microsoft.com/previous-versions/azure/windows-server-azure-pack/mt125371(v=technet.10)>)
- [Python Example by Microsoft](https://learn.microsoft.com/visualstudio/python/configure-web-apps-for-iis-windows?view=vs-2022#configure-the-httpplatform-handler)
- [Python/Flask Example by me]({% post_url 2022/2022-7-10-running-flask-web-apps-on-iis-with-httpplatformhandler %})
- [Python/Django Example by me]({% post_url 2024/2024-3-30-running-django-web-apps-on-iis-with-httpplatformhandler %})
- [Python/FastAPI Example by me]({% post_url 2024/2024-3-30-running-fastapi-web-apps-on-iis-with-httpplatformhandler %})
- [Node.js Example by me]({% post_url 2022/2022-6-11-running-nodejs-web-apps-on-iis-with-httpplatformhandler %})
- [Next.js Example by me]({% post_url 2024/2024-3-31-running-next-js-web-apps-on-iis-with-httpplatformhandler %})
- [Nuxt 3 Example by me]({% post_url 2023/2023-1-16-running-nuxt-3-web-apps-on-iis-with-httpplatformhandler %})
- [Nest Example by me]({% post_url 2024/2024-4-5-running-nest-web-apps-on-iis-with-httpplatformhandler %})
- [Ruby Example by Scott Hanselman](https://www.hanselman.com/blog/announcing-running-ruby-on-rails-on-iis8-or-anything-else-really-with-the-new-httpplatformhandler)
- [Go Example by me]({% post_url 2024/2024-2-9-running-go-web-apps-on-iis-with-httpplatformhandler %})
