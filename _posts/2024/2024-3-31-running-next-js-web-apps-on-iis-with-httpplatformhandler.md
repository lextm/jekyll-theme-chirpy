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

## Prerequisites

To follow this post, you need to have the following software installed,

* Windows 10 or Windows Server 2016 or later (IIS 10 or later)
* HttpPlatformHandler v1.2 (from Microsoft) or [v2.0 (from LeXtudio)]({% post_url 2024/2024-4-8-httpplatformhandler-v2 %})

## Basic Next.js Setup

First, use `create-next-app` to create a new Next.js project,

``` bash
$ cd C:\
$ mkdir test-nextjs
$ cd test-nextjs
$ npx create-next-app@latest
```

If you choose every option by default, you get a simple Next.js web app in `my-app` folder.

Then to run this Next.js web apps locally, you know you can use `npm run start` command.

``` bash
$ cd my-app
$ npm install
$ npm run build
$ npm run start

> my-app@0.1.0 start
> next start

   ▲ Next.js 14.1.4
   - Local:        http://localhost:3000

 ✓ Ready in 749ms
```

> Note that `npm run start` actually called `next start`, and by default the development server starts on port 3000.

## HttpPlatformHandler Setup

When you want to host the web app on IIS, you just need to create a `c:\test-nextjs\my-app\web.config` as below and invoke `next start` properly,

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <handlers>
            <add name="httpPlatformHandler" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" requireAccess="Script" />
        </handlers>
        <httpPlatform stdoutLogEnabled="true" stdoutLogFile=".\node.log" startupTimeLimit="20" processPath="C:\Users\<user name>\AppData\Roaming\nvm\v18.20.1\node.exe" arguments=".\node_modules\next\dist\bin\next start">
            <environmentVariables>
                <environmentVariable name="PORT" value="%HTTP_PLATFORM_PORT%" />
                <environmentVariable name="NODE_ENV" value="Production" />
            </environmentVariables>
        </httpPlatform>
    </system.webServer>
</configuration>
```

> Here I assume you use `nvm` to manage your Node.js versions and the active version is v18.20.1.

Now go to IIS Manager and create a new site to point to `C:\test-nextjs\my-app` folder with site binding on localhost port 8015. Then you can browse the site and see the Next.js web app running.

## Troubleshooting

If you encounter any issues, you can check the troubleshooting tips I wrote in [my previous post on Node.js]({% post_url 2022/2022-6-11-running-nodejs-web-apps-on-iis-with-httpplatformhandler %}).

## Side Notes

### Deployment Artifacts
You must run `npm run build` (wrapper over `next build`) to generate the artifacts before deploying to IIS. Besides, you can delete source files and only leave `.next`, `node_modules` and `web.config` in the deployment folder.

> It is possible to delete more bits from `node_modules` to reduce the deployment size, but that's another topic.

### Dynamic Routes

IIS and HttpPlatformHandler support dynamic routes out of the box. You don't need to do anything special to make them work.

### Next.js as IIS Application under a Site

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

## Other Languages on IIS?

If you want to learn more about HttpPlatformHandler and how to host other languages (Go/Python/Java) or frameworks (Nuxt.js), you can read [this post]({% post_url 2023/2023-4-7-the-rough-history-of-iis-httpplatformhandler %}).
