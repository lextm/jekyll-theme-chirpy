---
layout: post
title: "Running Nuxt 3 Web Apps on IIS with HttpPlatformHandler"
tags: IIS Windows JavaScript
excerpt_separator: <!--more-->
---

When Microsoft developed HttpPlatformHandler more than a decade ago to enable non-Microsoft web technologies on Windows/IIS, they didn't know that one day

* Microsoft can embrace Linux in Azure
* Some Microsoft users stick to IIS with their Java/Python/Node.js/Go applications.

Thus, HttpPlatformHandler still plays an important role in the ecosystem and won't go away easily. However, the landscape keeps evolving so this post tries to capture some latest changes on Nuxt 3 and show you how to proper set up everything needed and more critically how to troubleshoot if issues occur.
<!--more-->

# Sample Project Preparation

Compared to Nuxt 2.x releases, 3.0 introduced brand new steps so you must stick to [the official guide](https://nuxt.com/docs/getting-started/installation#new-project) closely,

``` bash
npx nuxi init test-nuxt
cd test-nuxt
npm install
npm run build
```

> Note that I chose `npx` and `npm` steps, while you can use `pnpm` or `yarn`.

# Add IIS Configuration

Simply create a `web.config` file at the root,

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <handlers>
            <add name="httpPlatformHandler" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" requireAccess="Script" />
        </handlers>
        <httpPlatform stdoutLogEnabled="true" stdoutLogFile=".\node.log" startupTimeLimit="20" processPath="C:\Users\<user name>\AppData\Roaming\nvm\v16.13.2\node.exe" arguments=".output\server\index.mjs">
            <environmentVariables>
                <environmentVariable name="PORT" value="%HTTP_PLATFORM_PORT%" />
                <environmentVariable name="NODE_ENV" value="Production" />
            </environmentVariables>
        </httpPlatform>
    </system.webServer>
</configuration>
```

With all settings in place, I can go back to IIS Manager and create a site (I chose *:8030 as site binding, but as a normal IIS site you can configure any bindings you like) to point to `C:\test-nuxt`. By opening a web browser and navigate to `http://localhost:8030/`, I can see "Welcome to Nuxt" page as expected.

If you are not familiar with the contents and hit any IIS error, please read [my previous post](/running-nodejs-web-apps-on-iis-with-httpplatformhandler/) to learn how to troubleshoot.
