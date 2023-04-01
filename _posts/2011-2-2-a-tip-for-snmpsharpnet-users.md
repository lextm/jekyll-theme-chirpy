---
layout: post
title: "A Tip for SnmpSharpNet (SNMP#NET) Users"
description: "This post is about a tip for SnmpSharpNet (SNMP#NET) users."
tags: SNMP
permalink: /a-tip-for-snmpsharpnet-snmp-net-users-6a23a02b71e
excerpt_separator: <!--more-->
---
Well, I believe you have the habit of reading release notes and pay attention to what was written in SnmpSharpNet's one.

http://www.snmpsharpnet.com/node/57

Then you should know that in almost all cases Dispose or using must be used for UdpTarget objects. However, many users ignored this tip and experienced socket problems (http://topic.csdn.net/u/20110114/15/7871705d-c71d-4e27-b01c-bf10fd108e07.html).

Luckily #SNMP (http://sharpsnmplib.codeplex.com) uses a different design which is free of such issues, so you can feel free to try it out.
<!--more-->