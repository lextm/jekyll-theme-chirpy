---
layout: post
title: The Rough History of ASP.NET on IIS
description: This post is about the rough history of ASP.NET on IIS.
tags: Visual-Studio .NET
permalink: /the-rough-history-of-asp-net-on-iis-8f49e2bcefcd
excerpt_separator: <!--more-->
---

> It all happened when I tried to answer [this Stack Overflow question](https://stackoverflow.com/questions/35639205/what-is-kestrel-vs-iis-express/46878663#46878663).

Scott Guthrie joined Microsoft in 1997 and started to build something that enables rapid web application development. He mentioned more technical details in several occations that the initial project was written in several programming languages (including Java) as experiments, and when .NET Framework/C# was selected to be the new platform for Windows development inside Microsoft, the framework he worked on was rewritten in C# and included as ASP.NET.
<!--more-->

> This version of ASP.NET introduced the HTTP request processing pipeline (classes around `HttpApplication`) as well as the UI control framework, which later is called WebForms (when ASP.NET MVC was introduced).

Two pieces of important software were required by ASP.NET developers,

* Cassini, later became ASP.NET Development Server in Visual Studio. It is a fully managed web server written in C# based on `HttpListener`. Of course, since it was designed for development only, many features were never implemented.
  
  > As Microsoft made the source code of Cassini available for the public, there are third parties who forked the code base and added more features, which started the Cassini family.

* ASP.NET support on IIS (revision 1). Because IIS was 4.0 and 5.0/5.1 at that time, which has nothing like application pools, ASP.NET even has its own worker process (`aspnet_wp.exe`), that hosts ASP.NET's own processing pipeline outside of IIS's pipeline.

So web apps were developed on Cassini, and then deployed to IIS.

> The war of web browsers was just over (IE beat Netscape), but the war of web servers continued, so naturally ASP.NET became a selling point for IIS.

* The introduction of application pools in IIS 6 required some changes on ASP.NET side, so `aspnet_wp.exe` became obsolete and replaced by `aspnet_isapi.dll`. That can be seen as ASP.NET support on IIS revision 2. So ASP.NET apps are being hosted in IIS worker processes (`w3wp.exe`).
* The introduction of integrated pipeline in IIS 7 and above required further changes, which replaced `aspnet_isapi.dll` with `webengine4.dll`. That can be seen as ASP.NET support on IIS revision 3. ASP.NET and IIS pipelines are unified.

> Microsoft documented the pipeline modes in [this article](https://docs.microsoft.com/iis/application-frameworks/building-and-running-aspnet-applications/aspnet-integration-with-iis#aspnet-integration-architecture).

WCF and ASP.NET MVC were introduced to enrich web apps/services development, but they also make the web stack complicated and monolithic. In that situation, Cassini started to show its age (and other problems), and was gradually replaced by IIS Express (a user mode lite IIS).

When ASP.NET Web API and SignalR were developed, Microsoft engineers started to develop the OWIN based pipeline and utilize the middleware concepts to decouple different components of the web framework. That attempt worked quite well for Web API and SignalR scenarios and enabled cool things like self hosting. However, later they found it difficult to migrate WebForms and MVC to this new pipeline without breaking compatibility. Therefore, a platform upgrade was unavoidable.

Then in November 2014, ASP.NET 5 (later renamed to ASP.NET Core) was announced and became a cross platform technology. Obviously Microsoft cannot bind it to IIS any more. So the new design must consider macOS and Linux, where nginx/Apache or other web servers should be used.

I think many would agree that Microsoft learned quite a lot from Node.js, and then designed and developed Kestrel (based on `libuv` initially but might move to other technology soon). It is a light weighted web server like Cassini initially, but later more features are being added (so can be treated as a full web server). Though fully managed (some native dependencies exist), it is no longer a toy web server like Cassini.

Then why cannot you just use Kestrel? Why IIS Express and potentially IIS, nginx, or Apache are still needed? That primarily is a result of today's internet practice. Most web sites use reverse proxies to take requests from your web browsers and then forward to the application servers in the background.

* IIS Express/IIS/nginx/Apache are the reverse proxy servers
* Kestrel/Node.js/Tomcat and so on are the application servers

[Microsoft has its documentation here](https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel).

Microsoft developed HttpPlatformHandler initially to make IIS a good enough reverse proxy for Java/Python and so on, so planned to use it for ASP.NET Core. Issues started to appear during development, so later Microsoft made ASP.NET Core Module specifically for ASP.NET Core. That's ASP.NET support on IIS revision 4.

Well, quite lengthy, but I hope I put all necessary pieces together and you enjoy reading it.

> Look for other interesting posts like this one? You can visit [the index page](/all-in-one-for-the-legends-of-net-materials-43c374a01433).
