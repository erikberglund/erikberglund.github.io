---
title:  "Script Tip: Get the currently logged in user, in Bash"
date:   2018-10-02 09:00:00
description: A native method for getting the currently logged in user.
---

Recently one of my postinstall scripts failed for a tiny subset of our users. It turned out to be the Python binary in their **PATH** that did not have the Python Objective-C bridge, so the function I used to get the currently logged in user failed.

That python snippet was written by the excellent [Michael Lynn](http://michaellynn.github.io/about/) and spread far and wide by the equally excellent [Ben Toms](https://macmule.com/about/) in his infamous post:

[How To: Get the currently logged in user, in a more Apple approved way](https://macmule.com/2014/11/19/how-to-get-the-currently-logged-in-user-in-a-more-apple-approved-way/).

I fixed my postinstall with a hardcoded path to the system Python, but I began to wonder: 

_Do I have to use Python to get this information reliably?_

# SystemConfiguration framework

The Python snippet is following the example in a Q&A article from Apple about getting the current console user: [Q&A1133: Determining console user login status](https://developer.apple.com/library/archive/qa/qa1133/_index.html). 

It's calling the function: [SCDynamicStoreCopyConsoleUser](https://developer.apple.com/documentation/systemconfiguration/1517123-scdynamicstorecopyconsoleuser) from the [SystemConfiguration framework](https://developer.apple.com/documentation/systemconfiguration) to read from the dynamic store maintained by [configd](x-man-page://8/configd) which contains data reflecting the current state of the system.

Since there are no Objective-C bridges to Bash we cannot use the APIs directly, but we already have a tool that uses these APIs to access the system configuration database.

# scutil

The binary **/usr/sbin/scutil** has a limited set of command line options, but from it's [man]((x-man-page://8/scutil)) page:

```string
Invoked with no options, scutil provides a command line interface to the
"dynamic store" data maintained by configd(8). 

Interaction with this data (using the SystemConfiguration.framework SCDynamicStore APIs) is handled with a set of commands read from standard input.
```

First, we list the available keys in the data store by passing the interactive command **list**:

```bash
scutil <<< "list"
```

(The command above is using a [here string](https://www.tldp.org/LDP/abs/html/x17837.html) (**\<\<\<**) to pass a string to stdin)

This prints a list of the available keys. subKey 100 seems to be what we are looking for:


```bash
subKey [0] = Plugin:IPConfiguration
subKey [1] = Plugin:InterfaceNamer
subKey [2] = Plugin:KernelEventMonitor
subKey [3] = Setup:
subKey [4] = Setup:/
subKey [5] = Setup:/Network/BackToMyMac
subKey [6] = Setup:/Network/BackToMyMacDSIDs
...
subKey [100] = State:/Users/ConsoleUser
...
```

Now we run the interactive command **show key ["pattern"]** with our key:

```bash
scutil <<< "show State:/Users/ConsoleUser"
```

And we get what we came for:

```bash
<dictionary> {
  GID : 20
  Name : erikberglund
  SessionInfo : <array> {
    0 : <dictionary> {
      kCGSSessionAuditIDKey : 100008
      kCGSSessionGroupIDKey : 20
      kCGSSessionIDKey : 257
      kCGSSessionLoginwindowSafeLogin : FALSE
      kCGSSessionOnConsoleKey : TRUE
      kCGSSessionOrderingKey : 0
      kCGSSessionSystemSafeBoot : FALSE
      kCGSSessionUserIDKey : 501
      kCGSSessionUserNameKey : erikberglund
      kCGSessionLoginDoneKey : TRUE
      kCGSessionLongUserNameKey : Erik Berglund
      kSCSecuritySessionID : 100008
    }
  }
  UID : 501
}
```

This prints three key/value pairs for the current console user:

```bash
GID : 20
Name : erikberglund
UID : 501
```

And an array of all current GUI sessions:

```
SessionInfo : <array> {
    0 : <dictionary> {
      kCGSSessionAuditIDKey : 100008
      kCGSSessionGroupIDKey : 20
      kCGSSessionIDKey : 257
      kCGSSessionLoginwindowSafeLogin : FALSE
      kCGSSessionOnConsoleKey : TRUE
      kCGSSessionOrderingKey : 0
      kCGSSessionSystemSafeBoot : FALSE
      kCGSSessionUserIDKey : 501
      kCGSSessionUserNameKey : erikberglund
      kCGSessionLoginDoneKey : TRUE
      kCGSessionLongUserNameKey : Erik Berglund
      kSCSecuritySessionID : 100008
    }
  }
```

With this, we can create a simple Bash snippet to get the currently logged in user in an Apple approved way without shelling out to Python in our Bash scripts.

**EDIT: Since I posted this, user @tulgeywood on the MacAdmins Slack made a more concise version of the awk part of the command. I have updated this post to use his version.**

```bash
loggedInUser=$( scutil <<< "show State:/Users/ConsoleUser" | awk '/Name :/ && ! /loginwindow/ { print $3 }' )
```

(And it's quite a lot faster)

# Trust, but verify

We need to verify that this command will provide the same result as the Python snippet.

And since Apple was kind enough to provide the source for **scutil** and **configd** on [opensource.apple.com](https://opensource.apple.com/source/configd/configd-963.30.1/) it was easy to do.

There, we find the command "show" will run an internal function called **do_show**:

[commands.c](https://opensource.apple.com/source/configd/configd-963.30.1/scutil.tproj/commands.c.auto.html)
```c
/* cmd    minArgs maxArgs func     group ctype usage*/
...
{ "show", 1,      2,      do_show, 4,    0,    " show key [\"pattern\"]..." }
...
```

**do_show** calls [SCDynamicStoreCopyValue](https://developer.apple.com/documentation/systemconfiguration/1437812-scdynamicstorecopyvalue?language=objc) with our key **"State:/Users/ConsoleUser"**:


[cache.c](https://opensource.apple.com/source/configd/configd-963.30.1/scutil.tproj/cache.c.auto.html)
```c
void
do_show(int argc, char **argv)
{
    ...
    newValue = SCDynamicStoreCopyValue(store, key);
    ...
    SCPrint(TRUE, stdout, CFSTR("%@\n"), newValue);
    CFRelease(newValue);
    return;
}
```

But **SCDynamicStoreCopyValue** != **SCDynamicStoreCopyConsoleUser**. 

So let's look at the **SCDynamicStoreCopyConsoleUser** function:

[SCDConsoleUser.c](https://opensource.apple.com/source/configd/configd-963.30.1/SystemConfiguration.fproj/SCDConsoleUser.c.auto.html)
```c
CFStringRef
SCDynamicStoreCopyConsoleUser(SCDynamicStoreRef store, uid_t *uid, gid_t *gid) {
    CFStringRef        consoleUser    = NULL;
    CFDictionaryRef        dict        = NULL;
    CFStringRef        key;

    key  = SCDynamicStoreKeyCreateConsoleUser(NULL);
    dict = SCDynamicStoreCopyValue(store, key);
    ...
    consoleUser = CFDictionaryGetValue(dict, kSCPropUsersConsoleUserName);
    ...
    return consoleUser;
}
```

There we can se it's also calling **SCDynamicStoreCopyValue** to get the value, using the key returned from **SCDynamicStoreKeyCreateConsoleUser(NULL)**. 

And running that command returns the same key as we use in **scutil**:

```objc
NSLog(@"%@", SCDynamicStoreKeyCreateConsoleUser(NULL));
State:/Users/ConsoleUser
```

This verifies that the **scutil** command uses the exact same API-call to get the console user.

# Where does the value come from?

After looking at this, I got curious about where the SystemConfiguration got its data.

That finally brought me to **loginwindow.app**.

In the initialization of the Login1 class, it's setting up callbacks for CoreGraphics notifications.

(All code snippets below are [Hopper](https://www.hopperapp.com) pseudo-code)

```
/* @class Login1 */
-(void *)init {
    var_50 = self;
    rcx = *0x100117e48;
    *(&var_50 + 0x8) = rcx;
    rax = [[&var_50 super] init];
    r12 = rax;
    ...
    CGSRegisterNotifyProc(sub_100032528, 0x5dc, r12);
    ...
```

With **CGSRegisterNotifyProc** it registers the function **sub_100032528** to be called whenever a notification for the **CGSEventType:** **0x5dc** is recieved.

**0x5dc** is hexadecimal for 1500 and it's defined as:

```
kCGSessionConsoleConnect = 1500
``` 

So whenever this notification is posted, **sub_100032528** will be run:

```
int sub_100032528(int arg0, int arg1, int arg2, int arg3) {
  rcx = arg3;
  r14 = rcx;
  syslog$DARWIN_EXTSN(0x3, "Session ON console", arg2, rcx);
  ...
  [[LoginDServer sharedLoginDServer] setSessionHasConsoleAccessFlag:0x1];
  if ([*qword_10011a9f0 loggedIn] == 0x1) {
    if (sub_1000416a1() == 0x1) {
      r15 = CGSSessionCopyAllSessionProperties();
        if (r15 != 0x0) {
          [[LoginDServer sharedLoginDServer] setSCDynamicStoreConsoleInformation:[r14 shortName] uid:[r14 userID] gid:[r14 groupID] sessionList:r15];
          ...
```

It checks that the user is logged in, then calls **setSCDynamicStoreConsoleInformation**:

```c
setSCDynamicStoreConsoleInformation:uid:gid:sessionList:
```

Which in turn calls **SCDynamicStoreSetConsoleInformation** to update the SystemConfiguration dynamic store with the current user information.

```c
SCDynamicStoreSetConsoleUser (    SCDynamicStoreRef store, 
                const char *user, 
                uid_t uid, 
                gid_t gid );
```

We could go on here diving into CoreGraphics and the **SkyLight.framework** that handles the GUI sessions and the WindowServer. But I will stop here not to miss the point of this post.

# Another Bash Method

There is another dynamic store we can read to get the console user in Bash: [The I/O Registry](https://developer.apple.com/library/archive/documentation/DeviceDrivers/Conceptual/IOKitFundamentals/TheRegistry/TheRegistry.html).

From the description, it doesn't seem to hold user information, but in the global **IORegistryEntry** root object there is a key called **IOConsoleUsers** that holds the actual array shown in the `SessionInfo` key from the **scutil** command.

That means that we can use **/usr/sbin/ioreg** to read the registry and find the dictionary with key **kCGSSessionOnConsoleKey** set to **TRUE** and print that username:

```bash
loggedInUser=$( ioreg -n Root -d1 -a | xpath '/plist/dict/key[.="IOConsoleUsers"]/following-sibling::array/dict/key[.="kCGSSessionOnConsoleKey"]/following-sibling::*[1][name()="true"]/../key[.="kCGSSessionUserNameKey"]/following-sibling::string[1]/text()' 2>/dev/null )
```

This information should be as valid as the one using the **scutil** method as it's the CoreGraphics framework **SkyLight** that writes this information before notifying the login window.

# Get the current console user in Bash

Here is the bash snippet again to get the current console user in Bash:

**EDIT: Since I posted this, user @tulgeywood on the MacAdmins Slack made a more concise version of the awk part of the command. I have updated this post to use his version.**

```bash
loggedInUser=$( scutil <<< "show State:/Users/ConsoleUser" | awk '/Name :/ && ! /loginwindow/ { print $3 }' )
```

And below is my original command. Both return the exact same information but the command above is just a bit shorter and easier to read, so use that.

```bash
loggedInUser=$( scutil <<< "show State:/Users/ConsoleUser" | awk -F': ' '/[[:space:]]+Name[[:space:]]:/ { if ( $2 != "loginwindow" ) { print $2 }}' )
```