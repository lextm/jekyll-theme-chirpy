---
layout: post
title: "If You Cannot Install/Uninstall IIS 7, Part II"
description: "This post is about how to fix IIS 7 installation issues."
tags: Windows IIS
permalink: /if-you-cannot-install-uninstall-iis-7-part-ii-73a14fe3db0
excerpt_separator: <!--more-->
---
I blogged about this last year as [part I](/if-you-cannot-install-uninstall-iis-7-292f3d582837).

In this part, I am going to tell you more about the corruption.
<!--more-->

Microsoft releases a tool for you to detect CBS (component based setup) issues, named System Update Readiness Tool,

http://windows.microsoft.com/en-US/windows7/What-is-the-System-Update-Readiness-Tool

You can download the one that matches your system and then execute it. Note that this tool was designed to fix some of Windows Update issues, so it does not only report CBS issue details, but also can resolve some CBS issues.

After executing this tool, it will generate a log file here,

%windir%\Logs\CBS\CheckSur.log

By opening this file, you can see what has been broken and attempt to fix that,

http://support.microsoft.com/kb/2700601

I suggest once found errors in this log file you open a support case via http://support.microsoft.com to ask for assistance. I attempted to resolve the errors on my box, but could not resolve them all manually.

Anyway, an in-place Windows upgrade usually resolves all issues, which is what I did last time,

http://support.microsoft.com/kb/2255099

Note that this is simpler than completely reinstalling Windows on the box.

(Updated: Microsoft finally published a similar article here to summarize the steps, https://docs.microsoft.com/en-us/iis/troubleshoot/installation-issues/troubleshooting-iis-7x-installation-issues).
