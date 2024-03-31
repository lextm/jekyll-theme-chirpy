---
layout: post
title: "Running Django Web Apps on IIS with HttpPlatformHandler"
description: A post about how to create a simple Django web app and host it on IIS with HttpPlatformHandler
tags: IIS Windows Python Django HttpPlatformHandler
excerpt_separator: <!--more-->
---

HttpPlatformHandler can help IIS host Java/Python/Node.js/Go applications, so in this post we wil see how to configure a Python/Django web app on IIS and troubleshoot the common issue.

It becomes very important for Python developers to learn HttpPlatformHandler, because [Microsoft no longer recommends FastCGI](https://docs.microsoft.com/visualstudio/python/configure-web-apps-for-iis-windows?view=vs-2022#configure-the-fastcgi-handler),

> "We recommend using HttpPlatform to configure your apps, as the WFastCGI project is no longer maintained."

<!--more-->

## Basic Django Setup

No doubt we will start from a sample application as below.

``` bash
cd C:\test-django
C:\Users\lextudio\AppData\Local\Programs\Python\Python310\python.exe -m django startproject mysite
`

This generates a Django project in `C:\test-django\mysite`, and now we can use a simple command `python.exe .\mysite\manage.py runserver` in the directory of `C:\test-django\` can launch the application at port 8000,

``` text
C:\test-django>C:\Users\lextudio\AppData\Local\Programs\Python\Python310\python.exe .\mysite\manage.py runserver  
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
March 30, 2024 - 21:58:13
Django version 5.0.3, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

> If Python and Django are not yet installed, you can search for guides.

## HttpPlatformHandler Setup

Now let's download and install HttpPlatformHandler on IIS, and add a `web.config` in `C:\test-django`,

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <handlers>
            <add name="httpPlatformHandler" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" requireAccess="Script" />
        </handlers>
        <httpPlatform stdoutLogEnabled="true" stdoutLogFile=".\python.log" startupTimeLimit="20" processPath="C:\Users\<user name>\AppData\Local\Programs\Python\Python310\python.exe" arguments=".\mysite\manage.py runserver %HTTP_PLATFORM_PORT%">
        </httpPlatform>
    </system.webServer>
</configuration>
```

With all settings in place, I can go back to IIS Manager and create a site (I chose *:8014 as site binding) to point to `C:\test-django`. By opening a web browser and navigate to `http://localhost:8014/`, I should now see the default Django welcome page saying "The install worked successfully! Congratulations!".

## Troubleshooting

Well if you hit any error instead of the JSON response, please refer to [my old post]({% post_url 2022/2022-7-10-running-flask-web-apps-on-iis-with-httpplatformhandler %}) for troubleshooting tips.

### Django on IIS Express

I also created the necessary PowerShell scripts to help you enable HttpPlatformHandler on IIS Express if you want to give it a try. You can find them on [my GitHub repository](https://github.com/lextm/iisexpress-httpplatformhandler).
