---
layout: post
title: "Running Node.js Web Apps on IIS with HttpPlatformHandler"
tags: IIS Windows JavaScript
excerpt_separator: <!--more-->
---

When Microsoft developed HttpPlatformHandler more than a decade ago to enable non-Microsoft web technologies on Windows/IIS, they didn't know that one day

* Microsoft can embrace Linux in Azure
* Some Microsoft users stick to IIS with their Java/Python/Node.js/Go applications.

Thus, HttpPlatformHandler still plays an important role in the ecosystem and won't go away easily. However, the landscape keeps evolving so this post tries to capture some latest changes on Node.js and show you how to proper set up everything needed and more critically how to troubleshoot if issues occur.
<!--more-->

# Basic Node.js Setup

No doubt we will start from a sample application as below,

``` javascript
import { createServer } from 'http';

const port = process.env.PORT || 3000;

const requestListener = function(req, res) {
  res.writeHead(200);
  res.end('My first server');
}

const server = createServer(requestListener);
server.listen(port, function () {
  console.log('Example app listening on port ' + port + '!');
});
```

If we save it as `C:\node-test\app.js`, then on a Windows machine with Node.js installed, a simple command `node app.js` can launch the application at port 3000,

``` text
PS C:\node-test> node app.js
Example app listening on port 3000!
```

# HttpPlatformHandler Setup

Now let's download and install HttpPlatformHandler on IIS, and add a `web.config` in `C:\node-test`,

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <handlers>
            <add name="httpPlatformHandler" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" requireAccess="Script" />
        </handlers>
        <httpPlatform stdoutLogEnabled="true" stdoutLogFile=".\node.log" startupTimeLimit="20" processPath="C:\Program Files\nodejs\node.exe" arguments=".\app.js">
            <environmentVariables>
                <environmentVariable name="PORT" value="%HTTP_PLATFORM_PORT%" />
                <environmentVariable name="NODE_ENV" value="Production" />
            </environmentVariables>
        </httpPlatform>
    </system.webServer>
</configuration>
```

With all settings in place, I can go back to IIS Manager and create a site (I chose *:8099 as site binding) to point to `C:\node-test`. By opening a web browser and navigate to `http://localhost:8099/`, I can see "My first server" as expected.

# Troubleshooting

## 0x80070002
But wait! Why did I see an error page saying "HTTP Error 502.3 - Bad Gateway" with Error Code `0x80070002`?

![img-description](/images/bad-gateway-80070002.png)
_Figure 1: Bad Gateway error page of 0x80070002_

If you are very familiar with Windows error code, then you know that this error indicates "file not found",

``` text
> .\Err.exe 80070002
# No results found for hex 0x4c5c572 / decimal 80070002
# for hex 0x80070002 / decimal -2147024894
  COR_E_FILENOTFOUND                                             corerror.h
  DIERR_NOTFOUND                                                 dinput.h
  DIERR_OBJECTNOTFOUND                                           dinput.h
  STIERR_OBJECTNOTFOUND                                          stierr.h
  DRM_E_WIN32_FILE_NOT_FOUND                                     windowsplayready.h
  E_FILE_NOT_FOUND                                               wpc.h
# as an HRESULT: Severity: FAILURE (1), FACILITY_NTWIN32 (0x7), Code 0x2
# for hex 0x2 / decimal 2
  STATUS_WAIT_2                                                  ntstatus.h
# as an HRESULT: Severity: FAILURE (1), FACILITY_WIN32 (0x7), Code 0x2
  ERROR_FILE_NOT_FOUND                                           winerror.h
# The system cannot find the file specified.
# 8 matches found for "80070002"
```

So which part of our settings can trigger this error? Luckily our first attempt on `C:\Program Files\nodejs\node.exe` seems to raise a red flag,

![img-description](/images/nodejs-windows.png)
_Figure 2: Node.js Windows shortcut_

As the image shows, the Node.js installer creates a shortcut here, so that `C:\Program Files\nodejs\node.exe` is in fact `C:\Users\<user name>\AppData\Roaming\nvm\v16.13.2\node.exe` on this machine.

> Note that the shortcut on your machine might point to a different location.

Due to the design of IIS, such shortcuts do not work very well, so we need to change `web.config` as below here,

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <handlers>
            <add name="httpPlatformHandler" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" requireAccess="Script" />
        </handlers>
        <httpPlatform stdoutLogEnabled="true" stdoutLogFile=".\node.log" startupTimeLimit="20" processPath="C:\Users\<user name>\AppData\Roaming\nvm\v16.13.2\node.exe" arguments=".\app.js">
            <environmentVariables>
                <environmentVariable name="PORT" value="%HTTP_PLATFORM_PORT%" />
                <environmentVariable name="NODE_ENV" value="Production" />
            </environmentVariables>
        </httpPlatform>
    </system.webServer>
</configuration>
```

> Note that to learn all settings such as request timeout, you can refer to [this article](https://docs.microsoft.com/iis/extensions/httpplatformhandler/httpplatformhandler-configuration-reference).

## 0x80070005

OK, now refreshing the browser gives us a different Bad Gateway error page because its Error Code changes to `0x80070005`,

![img-description](/images/bad-gateway-80070005.png)
_Figure 3: Bad Gateway error page of 0x80070005_

``` text
> .\Err.exe 80070005
# No results found for hex 0x4c5c575 / decimal 80070005
# for hex 0x80070005 / decimal -2147024891
  COR_E_UNAUTHORIZEDACCESS                                       corerror.h
# Access is denied.
  DIERR_OTHERAPPHASPRIO                                          dinput.h
  DIERR_READONLY                                                 dinput.h
  DIERR_HANDLEEXISTS                                             dinput.h
  DSERR_ACCESSDENIED                                             dsound.h
  STIERR_READONLY                                                stierr.h
  STIERR_NOTINITIALIZED                                          stierr.h
  E_ACCESSDENIED                                                 winerror.h
# General access denied error
# as an HRESULT: Severity: FAILURE (1), FACILITY_WIN32 (0x7), Code 0x5
# for hex 0x5 / decimal 5
  ERROR_ACCESS_DENIED                                            winerror.h
# Access is denied.
# 9 matches found for "80070005"
```

This also makes perfect sense, because anything under `C:\Users\<user name>\` is protected and accessible only by that user account by default, not `IIS_IUSRS`.

# The End
Once I grant `IIS_IUSRS` read access to `C:\Users\<user name>\AppData\Roaming\nvm\v16.13.2\node.exe`, the browser starts to work as expected and gives me "My first server".

With Process Explorer I can further analyze the `node.exe` process spin off by `w3wp.exe`,

![img-description](/images/node-process-iis.png)
_Figure 4: Node.js process under IIS_

And if an application pool recycle is triggered in IIS Manager, I can also observe the two processes being shut down peacefully.

> Note that the peaceful shutdown is via Windows API `OpenProcess(SYNCHRONIZE | PROCESS_TERMINATE, FALSE, m_dwProcessId)`. You can read [ASP.NET Core module source code](https://github.com/dotnet/aspnetcore/blob/v6.0.5/src/Servers/IIS/AspNetCoreModuleV2/OutOfProcessRequestHandler/serverprocess.cpp#L1337) to learn more, as this module is derived from HttpPlatformHandler.

# Side Notes

## Express for Node.js
One thing you might notice is that I wrote a very simple Node.js application as example. Why not go a little bit further to use a framework like Express for Node.js? In fact I did try it out (Express 4.18.1), but `node.exe` does not shut down peacefully and it also blocks `w3wp.exe` from proper shutdown. I didn't have time to dig further there, but you might find it an interesting field to explore. Good luck.

## Hosting on Azure App Service (Windows)
Two small changes might be needed if you want to deploy `app.js` and `web.config` together to your Azure App Service (Windows),

1. You might need to change its name from `app.js` to `app.mjs`, because Node.js 16+ might tell you that `SyntaxError: Cannot use import statement outside a module`. So that the value of `arguments` now should be `.\app.mjs`.
1. You might change the value of `processPath` to simply `node.exe` as Azure App Service seems to be able to resolve it to the correct executable.

The complete `web.config` file might look as below,

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <handlers>
            <add name="httpPlatformHandler" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" requireAccess="Script" />
        </handlers>
        <httpPlatform stdoutLogEnabled="false" stdoutLogFile=".\node.log" startupTimeLimit="20" processPath="node.exe" arguments=".\app.mjs">
            <environmentVariables>
                <environmentVariable name="PORT" value="%HTTP_PLATFORM_PORT%" />
                <environmentVariable name="NODE_ENV" value="Production" />
            </environmentVariables>
        </httpPlatform>
    </system.webServer>
</configuration>
```

## Nuxt.js

If you are working on a Nuxt.js project, make sure you modify it further before deploying to IIS. The [official guide for Azure Portal](https://nuxtjs.org/deployments/azure-portal) shows the general hints, 

1. Create `server\index.js`.
1. Modify `nuxt.config.js`.

but it misses important steps,

* You must add `express` and `nuxt-start` as dependencies (check your `package.json`).
* Change `const nuxt = await loadNuxt(isDev ? 'dev' : 'start')` to simply `const nuxt = await loadNuxt('start')`, as `isDev` isn't defined anywhere.

Then your `web.config` can work with HttpPlatformHandler as below,

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <handlers>
            <add name="httpPlatformHandler" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" requireAccess="Script" />
        </handlers>
        <httpPlatform stdoutLogEnabled="true" stdoutLogFile=".\node.log" startupTimeLimit="20" processPath="C:\Users\<user name>\AppData\Roaming\nvm\v16.13.2\node.exe" arguments=".\server\index.js">
            <environmentVariables>
                <environmentVariable name="PORT" value="%HTTP_PLATFORM_PORT%" />
                <environmentVariable name="NODE_ENV" value="Production" />
            </environmentVariables>
        </httpPlatform>
    </system.webServer>
</configuration>
```

> I didn't add the rewrite rules as the minimal sample project does not require them, but you can add them for your project if needed.
