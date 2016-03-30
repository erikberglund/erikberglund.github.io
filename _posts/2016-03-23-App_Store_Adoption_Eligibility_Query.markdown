---
title:  "App Store Adoption Eligibility Query"
description: Get computer app adoption status
date: 2016-03-23 20:00:00
---

When you buy a new computer from Apple, the following Mac App Store applications are pre-installed on the system without a tie to a particular Apple ID:

* iMovie
* GarageBand
* Pages
* Numbers
* Keynote

<br>
To update them, the user has to "adopt" the applications using an Apple ID. ([HT203658](https://support.apple.com/en-us/HT203658))

By doing that, the applications become associated with that Apple ID locally and the computer that's used to adopt the licenses has it's free licenses removed from the Apple servers.

This means that the apps can't be adopted by another Apple ID using the same computer.

# Adoption Eligibility

When the Mac App Store application launches, it reaches out to [buy.itunes.apple.com](http://buy.itunes.apple.com) which returns the status of the running machine's app adoption eligibility.

The server response is written to this file:

```console
/Library/Application Support/App Store/adoption.plist
```

If all applications have been adopted, the file will contain an error dict with the error key:

**MZFinance.AppAdoptionNotEligible**


```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>eligibilityRead</key>
    <dict>
        <key>errorMessage</key>
        <string>ex.name (com.apple.jingle.foundation.exceptions.MZCodedException) -- ex.message (MZException code = MZFinance.AppAdoptionNotEligible, userInfo = null)</string>
        <key>errorMessageKey</key>
        <string>MZFinance.AppAdoptionNotEligible</string>
        <key>errorNumber</key>
        <integer>3552</integer>
        <key>status</key>
        <integer>-1</integer>
        <key>userPresentableErrorMessage</key>
        <string>Future updates of these apps can be found through Software Update.</string>
    </dict>
    <key>eligiblityReadDate</key>
    <date>2016-03-23T20:00:00Z</date>
</dict>
</plist>
```

If one or more applications are available for adoption, an array of application bundle ID:s eligble for adoption will be returned instead.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>eligibilityRead</key>
    <dict>
        <key>items</key>
        <array>
            <dict>
                <key>version</key>
                <string>10.1</string>
                <key>adamID</key>
                <string>682658836</string>
                <key>installerVersionID</key>
                <string>811655132</string>
                <key>bundleID</key>
                <string>com.apple.garageband10</string>
            </dict>
            ...
        </array>
        <key>status</key>
        <integer>0</integer>
    </dict>
    <key>eligiblityReadDate</key>
    <date>2016-03-23T20:00:00Z</date>
</dict>
</plist>
```

So, to see the adoption status of a running computer, you only have to look in this file.

I've seen this file be empty on some systems. I have also seen instances when this query is never sent by the Mac App Store. In those cases running the query yourself is a good way to find out the computer's app adoption status.

# Deciphering the query

I want to check the adoption status from a NetBoot environment where running the Mac App Store to update the adoption.plist file can't be done.

Therefore we have to recreate the Mac App Store request ourselves.

By enabling xml debug logging for the Mac App Store using the following [**defaults**](x-man-page://1/defaults) command:

```console
defaults write com.apple.commerce LogXML true
```

We can see that it's the `storeassetd` binary that requests this information, and in fact, log it's queries in the following location:

```console
~/Library/Logs/storeassetd
```

Look for files named **_date_ (Request).plist** and inspect their content. We're looking for requests that have the URL key set to an URL looking like this:

```console
https://buy.itunes.apple.com/WebObjects/MZFinance.woa/wa/adoptionEligibilitySrv?guid=A45E60******
```

In this file, we can see (in the **httpBody** key) that the request data is a plist with four keys:

```xml
<dict>
    <key>guid</key>
    <string>A45E60******</string>
    <key>response-content-type</key>
    <string>text/xml</string>
    <key>rom-addr</key>
    <string>0cbc9f******</string>
    <key>serial-no</key>
    <string>C0252******GF2C1H</string>
</dict>
```

### guid

In this context, the **guid** is the MAC address (without separator and all upper-case) of the first interface (en0). In my testings this value is not required, it can be left empty.

You can use this command to print the MAC address for interface **en0** in the correct format:

```bash
ifconfig en0 | awk '/ether/{ gsub(":",""); print toupper($2)}'
```

_Output:_

```console
A45E60******
```

### response-content-type

This key set the format for the return value, we'll use the current value of **text/xml**

### serial-no

The serial-no value is a 17 character string that looks like a longer version of the standard computer serial number. That's because it's the [main logic board serial number](http://www.insanelymac.com/forum/topic/303073-pattern-of-mlb-main-logic-board/#entry2091468).

This value is stored in nvram and by knowing it's address (**4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:MLB**) we can use the [**nvram**](x-man-page://8/nvram) command to retrieve the serial number:

```bash
nvram 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:MLB | awk '{ print $NF }'
```

_Output:_

```console
C0252******GF2C1H
```

### rom-addr

The rom-addr is a MAC address for the main logic board that's used in conjunction with the main logic board serial number to uniquely identify a Mac.

This value is also stored in nvram, and by using it's address (**4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:ROM**) we can retrieve the rom-addr using the [**nvram**](x-man-page://8/nvram) command as well:

```bash
nvram -x 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:ROM | awk '{ gsub(/\%/, ""); print $NF }'
```

_Output:_

```console
0cbc9f******
```

# Query without the Mac App Store

Now that we know how to retrieve the required values, we can send a query to the Apple servers for the adoption status of the running system (must be saved as a bash script to run):

```bash
#!/bin/bash

guid=$(ifconfig en0 | awk '/ether/{ gsub(":",""); print toupper($2)}')

read -r -d '' data_plist <<-EOF
	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
	<dict>
		<key>guid</key>
		<string>${guid}</string>
		<key>response-content-type</key>
		<string>text/xml</string>
		<key>rom-addr</key>
		<string>$(nvram 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:ROM | awk '{ gsub(/\%/, ""); print $NF }')</string>
		<key>serial-no</key>
		<string>$(nvram 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:MLB | awk '{ print $NF }')</string>
	</dict>
	</plist>
	EOF

curl -H "X-Apple-Store-Front: 143441-1,12" --data "${data_plist}" https://buy.itunes.apple.com/WebObjects/MZFinance.woa/wa/adoptionEligibilitySrv?guid=${guid}

exit 0
```

This will return the same property list as the one written to adoption.plist

# Deciphering the response

The response will be in property list format (as defined by the **response-content-type** key) and can either contain error keys or a list of available applications to adopt.

### Error Response
If you get an error response, here are some keys that might be returned:

* **MZFinance.AppAdoptionNotEligible**  
 This means there are no applications to adopt for this computer (or they're already adopted).

* **MZFinance.NoSerialNos / MZCommerce.SoftwareAdoption.NoSerialNos**  
 This is returned if either the **serial-no** or **rom-addr** are incorrect, or if the computer never had any free application licenses to grant the user.

_Example response when all applications have been adopted:_

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>errorMessageKey</key>
    <string>MZFinance.AppAdoptionNotEligible</string>
    <key>errorNumber</key>
    <integer>3552</integer>
    <key>status</key>
    <integer>-1</integer>
    <key>userPresentableErrorMessage</key>
    <string>Future updates of these apps can be found through Software Update.</string>
    <key>errorMessage</key>
    <string>ex.name (com.apple.jingle.foundation.exceptions.MZCodedException) -- ex.message (MZException code = MZFinance.AppAdoptionNotEligible, userInfo = null)</string>
    </dict>
</plist>
```

### Adoption Response:

If you instead get a list of applications still available for adoption, it will look like this.

_Example response when multiple applications are available for adoption:_

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>eligibilityRead</key>
    <dict>
        <key>items</key>
        <array>
            <dict>
                <key>version</key>
                <string>10.1</string>
                <key>adamID</key>
                <string>682658836</string>
                <key>installerVersionID</key>
                <string>811655132</string>
                <key>bundleID</key>
                <string>com.apple.garageband10</string>
            </dict>
            <dict>
                <key>version</key>
                <string>10.1.1</string>
                <key>adamID</key>
                <string>408981434</string>
                <key>installerVersionID</key>
                <string>815650084</string>
                <key>bundleID</key>
                <string>com.apple.iMovieApp</string>
            </dict>
            <dict>
                <key>version</key>
                <string>6.6.1</string>
                <key>adamID</key>
                <string>409183694</string>
                <key>installerVersionID</key>
                <string>814208678</string>
                <key>bundleID</key>
                <string>com.apple.iWork.Keynote</string>
            </dict>
            <dict>
                <key>version</key>
                <string>3.6.1</string>
                <key>adamID</key>
                <string>409203825</string>
                <key>installerVersionID</key>
                <string>814208661</string>
                <key>bundleID</key>
                <string>com.apple.iWork.Numbers</string>
            </dict>
            <dict>
                <key>version</key>
                <string>5.6.1</string>
                <key>adamID</key>
                <string>409201541</string>
                <key>installerVersionID</key>
                <string>814208633</string>
                <key>bundleID</key>
                <string>com.apple.iWork.Pages</string>
            </dict>
        </array>
        <key>status</key>
        <integer>0</integer>
    </dict>
    <key>eligiblityReadDate</key>
    <date>2016-03-23T20:00:00Z</date>
</dict>
</plist>
```

# Final Notes

With this information I can query [buy.itunes.apple.com](http://buy.itunes.apple.com) directly from my NetBoot environment and act on the response for the running computer accordingly.

One action might be to populate the **.MASManifest** file at the path show below with the array from a positive response to have them show up in the Mac App Store as available for adoption on the imaged Mac. 

```console
/var/db/.MASManifest
```

Another might be to show a warning if a computer is expected to have all application adoptions remaining (in the case of demo units for example).