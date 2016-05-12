---
title:  "Extracting Payload Keys from Profile Manager"
date:   2016-05-12 17:00:00
description: Parse Profile Manager source code to output available payload keys.
---

I'm developing [ProfileCreator](https://github.com/ProfileCreator/ProfileCreator), an open source configuration profile creation tool. While doing this I've been looking at ways to automatically and repeatedly create a list of all available payload keys and their configuration for use in my app.

Apple publishes the [Configuration Profile Key Reference](https://developer.apple.com/library/ios/featuredarticles/iPhoneConfigurationProfileRef/Introduction/Introduction.html) at their developer site, but even though it's being updated, it doesn't contain <u>all</u> keys and configurations.

[Profile Manager](http://www.apple.com/osx/server/features/#profile-manager), in being Apple's own MDM product, is often the closest we have to a reference for OS X and iOS configuration profiles as it usually supports all new profile keys on their release. But as [Profile Manager](http://www.apple.com/osx/server/features/#profile-manager)'s profile creation is geared towards [OTA](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/iPhoneOTAConfiguration/Introduction/Introduction.html) deployment of profiles it enforces keys which aren't required if the profile is installed by other means.

[Apple Configurator 2](https://itunes.apple.com/us/app/apple-configurator-2/id1037126344?mt=12&ign-mpt=uo%3D2) is another application developed by Apple capable of creating configuration profiles (iOS only). But when you start comparing payload keys and options with [Profile Manager](http://www.apple.com/osx/server/features/#profile-manager) you see that some payloads have slightly different configuration options.

Neither of these applications have a reference that describe which payloads and keys they support so the only way to find that out is to create a profile, export it and inspect it's content.

# Finding the pieces

[Profile Manager](http://www.apple.com/osx/server/features/#profile-manager) is a web app with it's front end written in JavaScript and the back end in Ruby and PHP, which in turn also uses private Objective-C libraries.

_The information in this post comes from inspecting the Profile Manager source in Server 5.1_

This post will cover how to find and extract payload keys and their corresponding information from the source code instead of creating and saving profiles through it's GUI.

I'm also making a [python script](https://github.com/erikberglund/ProfileManagerKeyExtractor) available that I've put together while researching this that is currently able to extract most of the available payload information, but <u>it's not complete</u>.

The bulk of the payload information can be found in the front end code, so we'll start there.

# Front end

This is the path to the JavaScript fronted source code folder:

```console
.../Server.app/Contents/ServerRoot/usr/share/devicemgr/frontend/admin/common/app
```

Within is a JavaScript file named **javascript-packed.js** and two CSS-files. The JavaScript file (as the name implies) is _packed_ which is common for JavaScript code to save bandwidth. But this technique also obfuscates the file contents making it harder for humans to read. 

To unpack this file, I used [one](http://www.danstools.com/javascript-beautify/) of the many JavaScript "Beautifyer/Unpackers" online which expands and tries to structure the code in a human readable format.

Not only did the code expand from 1838 lines to 97302 lines, and the file size grew from 2965154 bytes to 4648523, it also made it possible for me to find out how the code is structured and find searchable patterns to single out payloads and their respective keys.

# KnobSet

"KnobSet" is the name given to payload groups that hold payload-specific property keys for a [PayloadType](https://developer.apple.com/library/ios/featuredarticles/iPhoneConfigurationProfileRef/Introduction/Introduction.html#//apple_ref/doc/uid/TP40010206-CH1-SW2). (There are some rare exceptions).

The variable **knobSetProperties** in the JavaScript source lists all available KnobSets 

_This example command only works on versions where the knobSetProperties list doesn't contain a newline_

```bash
sed -nE -e 's/.*knobSetProperties:\[([",a-zA-Z]+)\].*/\1/p' javascript-packed.js
```
Pass the _-l_/_--list_ flag to my [profileManagerKeyExtractor.py](https://github.com/erikberglund/ProfileManagerKeyExtractor) script for the same list:

```bash
./profileManagerKeyExtractor.py -l
```

With this list we can single out the relevant parts of the code that contain payload information for the selected KnobSet and parse it to get a structured output.

# Gathering the information

To access the payload information for a KnobSet we need the selected KnobSet object name.

_Example that extracts the object name for the calDavKnobSets property:_

```bash
sed -nE 's/.*,Admin.([a-zA-Z]+KnobSet).*profilePropertyName[=:]"calDavKnobSets".*/\1/p' javascript-packed.js
```

The payload keys available to a specific KnobSet is merged into the object using **extend()**. 

The **CalDavKnobSet** object contains information for keys, value type, default value(s) etc.

_Excerpt from the extend method for the CalDavKnobSet:_

```javascript
Admin.CalDavKnobSet = Admin.KnobSet.extend({
        accountDescription: SC.Record.attr(String, {
            key: "CalDAVAccountDescription",
            defaultValue: "_generic_string_My CalDAV Account".loc()
        }),
        hostName: SC.Record.attr(String, {
            key: "CalDAVHostName"
        }),
        portNumber: SC.Record.attr(Number, {
            key: "CalDAVPort",
            defaultValue: 8443
        }),
        principalUrl: SC.Record.attr(String, {
            key: "CalDAVPrincipalURL"
        }),
        ...
```

The same is true for the **KnobSetView** object which holds key descriptions, available selections, hint strings, default values etc.

_Excerpt from the extend method for the CalDavKnobSetView:_

```javascript
Admin.CalDavKnobSetView = Admin.KnobSetView.extend({
        contentView: Admin.KnobSetContentView.design({
            layout: {
                width: Admin.KNOB_WIDTH,
                centerX: 0,
                height: 420
            },
            childViews: ["accountDescriptionField", "hostNameField", "principalUrlField", "usernameField", "passwordField", "useSslField"],
            accountDescriptionField: Admin.KnobSetTextFieldAbsolute.design({
                layout: {
                    top: 5,
                    centerX: 0,
                    width: Admin.KNOB_WIDTH,
                    height: 70
                },
                label: "_generic_string_Account Description".loc(),
                description: "_generic_string_The display name of the account".loc(),
                fieldHint: "_generic_string_optional".loc(),
                contentBinding: ".owner.content",
                fieldContentValueKey: "accountDescription"
            }),
            hostNameField: Admin.KnobSetIpPortFieldViewAbsolute.design({
                layout: {
                    top: 75,
                    centerX: 0,
                    width: Admin.KNOB_WIDTH,
                    height: 70
                },
                label: "_generic_string_Account Hostname and Port"
                    .loc(),
                description: "_generic_string_The CalDAV hostname or IP address and port number".loc(),
                fieldHint: "_hint_required".loc(),
                contentBinding: ".owner.content",
                fieldIpHostContentValueKey: "hostName",
                fieldPortContentValueKey: "portNumber"
            }),
            ...
```

By knowing this, and by knowing the name of the KnobSet object we're interested in it's possible by using regular expression to extract the method content and subsequently parse that for the specific information we seek.

# Localization

Many of the string values in the source is localized, which means that the actual string you might get from a variable could look like this:

_Example from the CalDAV account description:_

```bash
"_generic_string_The display name of the account".loc()
```

As you can see, the string ends with the method .loc(). To get the string that's presented to the user we need to use the string we got as a variable name and extract that variable's value from the localization file. 

For English, the localization file can be found at the following path:

```console
.../Server.app/Contents/ServerRoot/usr/share/devicemgr/frontend/admin/en.lproj/\
app/javascript_localizedStrings.js
```

Then we just get the value for the **_generic\_string\_The display name of the account** variable:

```bash
sed -nE 's/"_generic_string_The display name of the account": (.*),$/\1/p' javascript_localizedStrings.js

"The display name of the account"
```

Or, if we use the same extraction from the french localization file:

```bash
"Nom d’affichage du compte"
```

# Back end

The information in the front end code is not complete. For example the PayloadType(s), if the payload allows multiple configurations (Unique) and the Payload Name is stored in ruby model files in the back end source code.

This is the path to the source folder for the back end ruby models:

```console
.../Server.app/Contents/ServerRoot/usr/share/devicemgr/backend/app/models
```

In this folder are all KnobSets separated into their own model files, so we can use the KnobSet object name to find the file containing a class that matches it and parse that file for info:

Example content of the **cal_dav_knob_set.rb** file (the file matching object _CalDavKnobSet_):

```ruby
class CalDavKnobSet < KnobSet

  @@payload_type          = "com.apple.caldav.account"
  @@payload_subidentifier = "caldav"
  @@is_unique             = false
  @@payload_name          = "CalDAV"
```

# Putting it together

With this knowledge, it's possible to parse the source to extract the payload information.

This is the current output from my [script](https://github.com/erikberglund/ProfileManagerKeyExtractor) if you request the information for the calDavKnobSet:

```console
$ ./profileManagerKeyExtractor.py -k calDavKnobSets

    Payload Name: CalDAV
    Payload Type: com.apple.caldav.account
          Unique: NO
       UserLevel: YES
     SystemLevel: NO
       Platforms: iOS,OSX

      PayloadKey: CalDAVAccountDescription
           Title: Account Description
     Description: The display name of the account
            Type: String
     Hint String: optional
    DefaultValue: My Calendar Account

      PayloadKey: CalDAVHostName
           Title: Account Hostname and Port
     Description: The CalDAV hostname or IP address and port number
            Type: String

      PayloadKey: CalDAVPort
           Title:
     Description:
            Type: Number
    DefaultValue: 8443

      PayloadKey: CalDAVPrincipalURL
           Title: Principal URL
     Description: The Principal URL for the CalDAV account
            Type: String

      PayloadKey: CalDAVUsername
           Title: Account User name
     Description: The CalDAV user name
            Type: String
     Hint String: Required (OTA)
                  Set on device (Manual)

      PayloadKey: CalDAVPassword
           Title: Account Password
     Description: The CalDAV password
            Type: String
     Hint String: Optional (OTA)
                  Set on device (Manual)

      PayloadKey: CalDAVUseSSL
           Title: Use SSL
     Description: Enable Secure Socket Layer communication with CalDAV server
            Type: Boolean
    DefaultValue: YES
```

# Final Notes

The method of parsing source code is of course **never** a good idea as any release might break the parsing. This was just a break from my work on [ProfileCreator](https://github.com/ProfileCreator/ProfileCreator) to see if it would be possible to create a tool that could be used to run on each [Profile Manager](http://www.apple.com/osx/server/features/#profile-manager) release and create a diff of what's changed so new additions would be easy to keep track of.

It was partly successful, and I can now manually get all information I need, but the script currently doesn't handle arrays or dicts as that would require some complex parsing logic.

Also, some strings that are returned cannot be found in the places I've looked and might be resolved in the private frameworks. 

I currently need to focus on building an alpha release of the [ProfileCreator](https://github.com/ProfileCreator/ProfileCreator) application, but when that is finished, I will probably revisit this script to improve it further. 

If anyone find this interesting please feel free to extend my script or use it as inspiration.
