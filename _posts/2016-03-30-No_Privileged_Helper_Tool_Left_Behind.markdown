---
title:  "No Privileged Helper Tool Left Behind"
description: Discovering orphaned privileged helper tools
date: 2016-03-30 20:00:00
---

With every release of OS X, Apple continues to improve and add new features to help developers in minimizing their application's [attack surface](https://en.wikipedia.org/wiki/Attack_surface). And as [privilege escalation](https://en.wikipedia.org/wiki/Privilege_escalation) is the base of most attacks, that is one of the most important vectors to keep patched and secure.

One of the many changes made to limit [privilege escalation attacks](https://en.wikipedia.org/wiki/Privilege_escalation) in OS X was the move to [XPC](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingXPCServices.html) as a way to attain [privilege separation](https://en.wikipedia.org/wiki/Privilege_separation) between processes used by an application.

[XPC](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingXPCServices.html) allows a developer to split an application into smaller components which are only granted the minimum rights required to perform their explicit function. See "[principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege)".

When done right, this removes the possibility for an attacker to gain full system rights if they manage to exploit one of the component processes running with restricted rights.

# Installing a Privileged Helper Tool

Before Snow Leopard, privilege escalation for an application was requested by the main application and granted to the requested tool by the system's [Security Server](https://developer.apple.com/library/mac/documentation/Security/Conceptual/Security_Overview/Architecture/Architecture.html).

But as of 10.7, [AuthorizationExecuteWithPrivileges](https://developer.apple.com/library/mac/documentation/Security/Reference/authorization_ref/#//apple_ref/c/func/AuthorizationExecuteWithPrivileges) was deprecated and since then all privilege escalation <u>should</u> (not required) be handled by a privileged helper tool.

> _As of Snow Leopard, this is the preferred method of managing privilege escalation on Mac OS X and should be used instead of earlier approaches such as BetterAuthorizationSample or directly calling [AuthorizationExecuteWithPrivileges](https://developer.apple.com/library/mac/documentation/Security/Reference/authorization_ref/#//apple_ref/c/func/AuthorizationExecuteWithPrivileges)  
>                                                                                          -- [SMJobBless sample project](https://developer.apple.com/library/mac/samplecode/SMJobBless/Introduction/Intro.html)_.

Typically, [SMJobBless](https://developer.apple.com/library/mac/documentation/ServiceManagement/Reference/ServiceManagement_header_reference/#//apple_ref/c/func/SMJobBless) is used to install the privileged helper tool on your system.

_Example prompt shown to the user when requesting to install the [AutoPkgr](https://github.com/lindegroup/autopkgr) helper tool:_

![SMJobBlessInstallPrompt]({{ site.url }}/assets/screenshots/SMJobBlessInstallPrompt.png)

If the user authenticates, a [launchd job](x-man-page://5/launchd.plist) is configured and copied here:

```console
/Library/LaunchDaemons/
```

The privileged helper tool binary is then copied from the main application bundle to:

```console
/Library/PrivilegedHelperTools/
```

The job is then registered and activated by [launchd](x-man-page://8/launchd).

# Removing a Privileged Helper Tool

The proper way (up until OS X 10.10 when it was [deprecated](https://developer.apple.com/library/mac/documentation/General/Reference/APIDiffsMacOSX10_10SeedDiff/frameworks/ServiceManagement.html)) was to call [SMJobRemove](https://developer.apple.com/library/mac/documentation/ServiceManagement/Reference/ServiceManagement_header_reference/index.html#//apple_ref/c/func/SMJobRemove) from the main application. Apple haven't provided an official replacement to that command, so I don't know what the officially sanctioned way to remove a privileged helper tool is.

But, the reason I'm writing this post is not for all the times a helper tool is properly removed.
 It's when the main application is removed and the helper tool is left on the system listening for connections on it's registered Mach Port.

# My Concern

A privileged helper tool, even a badly written one (containing an exploitable bug) is just as vulnerable to attacks with or without it's main application installed.

But here is the theoretical issue an orphaned privileged helper tool poses:

Say there is a new vulnerability discovered that targets the XPCService, protocol or any other code used in the communication or authentication of the helper tool.

In order to patch and update the tool, the developer have to release a new version of their application containing an updated helper tool and have the main application replace the potentially insecure one using [SMJobBless](https://developer.apple.com/library/mac/documentation/ServiceManagement/Reference/ServiceManagement_header_reference/#//apple_ref/c/func/SMJobBless).

This process can only be done through the main application, so a helper tool left on the system would not have any chance of receiving this update. The orphaned helper tool will continue to be launched by [launchd](x-man-page://8/launchd) and run with the old version until manually removed.

Even if this isn't a vulnerability in and of itself, I believe that this possible attack vector should be closed, as the helper tool doesn't serve any purpose if the main application is removed.

# Previous Attacks

One of the most notable exploits targeting an XPC service is by a fellow Swede, [Emil Kvarnhammar](https://twitter.com/emilkvarnhammar) of [TrueSec](http://www.truesec.se) who uncovered the RootPipe ([CVE-2015-1130](http://www.cvedetails.com/cve/CVE-2015-1130/)) vulnerability. 

If this topic interests you, here is a presentation from the TrueSec [Security Conference](http://www.securityconf.se) in June 2015 where Emil talks about the concepts i've touched on, and about RootPipe in particular: 

<iframe width="560" height="315" src="https://www.youtube.com/embed/cjgbPh_Freg" frameborder="0" allowfullscreen></iframe>

<br>

# Find PrivilegedHelperTools left behind

I've written a script to check for orphaned helper tools (currently requires OS X 10.10)

It's available on GitHub: [checkPrivilegedHelperToolStatus](https://github.com/erikberglund/Scripts/blob/master/tools/privilegedHelperToolStatus/checkPrivilegedHelperToolStatus)

It will inspect all binaries in **/Library/PrivilegedHelperTools** and print the following information:

* Helper tool info:  
    **Path**, **BundleID**, **Name**, **Version**

* Helper tool launchd job info:  
    **Path**, **Label**, **State** (running, waiting etc.)

* Rules the tool has registered in the authorization database.

* Applications registered to have access to the helper tool:  
    **Path**, **BundleID**, **Name**, **Version**

If the script couldn't find <u>any</u> application matching any of the application bundle id's registered to interact with the helper tool, a warning will be printed describing that this helper tool might be left behind and running without a main application.

_Example output where I've installed the [AutoPkgr](https://github.com/lindegroup/autopkgr) helper tool, then removed the application:_

```
        [Helper]
           Path: /Library/PrivilegedHelperTools/com.lindegroup.AutoPkgr.helper
                 
                 BundleID: com.lindegroup.AutoPkgr.helper
                 Name:     com.lindegroup.AutoPkgr.helper
                 Version:  1.3.6

       [Launchd]
           Path: /Library/LaunchDaemons/com.lindegroup.AutoPkgr.helper.plist
           
                 Label:    com.lindegroup.AutoPkgr.helper
                 State:    waiting
             
[Authorization DB]               
          Rules: com.lindegroup.autopkgr.remove.schedule.run
                 com.lindegroup.autopkgr.add.scheduled.run
                 com.lindegroup.autopkgr.pkg.uninstaller
                 com.lindegroup.autopkgr.uninstall.helper.tool
                 com.lindegroup.autopkgr.pkg.installer

  [Applications]
       BundleID: com.lindegroup.AutoPkgr
                 <No application with bundle id: 'com.lindegroup.AutoPkgr' found>
                 
                 +------------------------------------------------------+
                 |                    !!! WARNING !!!                   |
                 |                                                      |
                 |    NO APPLICATION FOUND FOR PRIVILEGED HELPER TOOL   |
                 |                                                      |
                 |               IT  MIGHT BE LEFT BEHIND               |
                 +------------------------------------------------------+
```

# Final Notes

The takeaway from this post should not be a fear of orphaned helper tools, but an awareness that they don't fill any other purpose than to be possibly exploited by a future vulnerability, or by a devloper mistake and then being left on a system without means to be patched.