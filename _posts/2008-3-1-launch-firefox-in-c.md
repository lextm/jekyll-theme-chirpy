---
layout: post
title: "Launch Firefox in C#"
tags: .NET
permalink: /launch-firefox-in-c-b74d7264d2a4
excerpt_separator: <!--more-->
---
I talked about this topic in [this post](/grapevine-voice-firefoxs-flaw-or-windows-e108ed376916), but today’s content is brand new. Yes, finally I find out how to launch Firefox without an exception. Then HOW?

The old code to launch Firefox was,

``` csharp
Process.Start("https://halfblood.pro");
```

And the new code is,

``` csharp
Help.ShowHelp(null, "https://halfblood.pro");
```

I don’t understand why Microsoft implements them differently and I don’t want to waste my time to dig .NET Framework source code under that MS-RL. Maybe the guys want to promote Internet Explorer, isn’t it?
<!--more-->
