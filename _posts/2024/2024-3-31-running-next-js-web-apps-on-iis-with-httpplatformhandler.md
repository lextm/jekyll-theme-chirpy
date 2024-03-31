---
layout: post
title: "Running Next.js Web Apps on IIS with HttpPlatformHandler"
description: A post about creating a simple Next.js web app and deploy it on IIS with HttpPlatformHandler
tags: IIS Windows JavaScript Node.js Next.js HttpPlatformHandler
excerpt_separator: <!--more-->
---

When Microsoft developed HttpPlatformHandler more than a decade ago to enable non-Microsoft web technologies on Windows/IIS, they didn't know that one day

* Microsoft can embrace Linux in Azure
* Some Microsoft users stick to IIS with their Java/Python/Node.js/Go applications.

Thus, HttpPlatformHandler still plays an important role in the ecosystem and won't go away easily. However, the landscape keeps evolving so this post tries to capture some latest changes on Next.js and show you how to proper set up everything needed and more critically how to troubleshoot if issues occur.
<!--more-->

### Basic Next.js Setup

First, use `create-next-app` to create a new Next.js project,

``` bash
npx create-next-app@latest
```

Then to run Next.js web apps locally, you know you can use `next start` command. 

## HttpPlatformHandler Setup

When you want to host the web app on IIS, you just need to create a `web.config` as below and invoke `next start` properly,

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <handlers>
            <add name="httpPlatformHandler" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" requireAccess="Script" />
        </handlers>
        <httpPlatform stdoutLogEnabled="true" stdoutLogFile=".\node.log" startupTimeLimit="20" processPath="C:\Users\<user name>\AppData\Roaming\nvm\v16.13.2\node.exe" arguments=".\node_modules\next\dist\bin\next start">
            <environmentVariables>
                <environmentVariable name="PORT" value="%HTTP_PLATFORM_PORT%" />
                <environmentVariable name="NODE_ENV" value="Production" />
            </environmentVariables>
        </httpPlatform>
    </system.webServer>
</configuration>
```

## Troubleshooting

If you encounter any issues, you can check the troubleshooting tips I wrote in [my previous post on Node.js]({% post_url 2022-6-11-running-nodejs-web-apps-on-iis-with-httpplatformhandler %}).

## Side Notes

You must run `next build` to generate the production artifacts before deploying to IIS. Besides, you can delete source files and only leave `.next`, `node_modules` and `web.config` in the deployment folder.

> It is possible to delete more bits from `node_modules` to reduce the deployment size, but that's another topic.

If you plan to host such a web app under a site as an IIS application (or subfolder, a term often used), you need to modify the `basePath` in your `next.config.mjs` file as below,

``` javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  basePath: '/AppWithNode1',
};

export default nextConfig;
```
where `AppWithNode1` is the name of the IIS application.

### Next.js on IIS Express

I also created the necessary PowerShell scripts to help you enable HttpPlatformHandler on IIS Express if you want to give it a try. You can find them on [my GitHub repository](https://github.com/lextm/iisexpress-httpplatformhandler).
