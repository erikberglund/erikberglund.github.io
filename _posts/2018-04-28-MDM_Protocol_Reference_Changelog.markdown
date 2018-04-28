---
title:  "Mobile Device Management Protocol Reference Changelog"
date:   2018-04-28 08:00:00
description: Version controlled mirror of Apple's MDM Protocol Reference
---

After my [last post](http://erikberglund.github.io/2018/Configuration_Profile_Reference_Changelog/) about my version control project of Apple's [Configuration Profile Reference](https://developer.apple.com/library/content/featuredarticles/iPhoneConfigurationProfileRef/Introduction/Introduction.html) I got a request from a fellow Mac admin and namesake [Eric](https://blog.eriknicolasgomez.com) about doing the same scraping and version control for the [Mobile Device Management Protocol Reference](https://developer.apple.com/library/content/documentation/Miscellaneous/Reference/MobileDeviceManagementProtocolRef).

I looked into this and found that it actually would be possible to create a scraper for any of these developer [Guides and Sample Code](https://developer.apple.com/library/content/navigation/) pages as they all have the same html formatting and structure.

I put all other projects on hold and developed a generic version of the scraping script against the [Mobile Device Management Protocol Reference](https://developer.apple.com/library/content/documentation/Miscellaneous/Reference/MobileDeviceManagementProtocolRef) page.

You can find my GitHub project for that page [here](https://github.com/erikberglund/Mobile-Device-Management-Protocol-Reference).

Unfortunately there were not as many archived pages for this page as it was private until sometime last year, but I managed to find a few pages to get a little peek into it's revision history.

The [Mobile Device Management Protocol Reference](https://developer.apple.com/library/content/documentation/Miscellaneous/Reference/MobileDeviceManagementProtocolRef) page is not really meant for [#macadmins](https://twitter.com/hashtag/macadmins) as it's a reference for MDM developers. But I still wanted to put it here for anyone insterested in the world of MDM.

Also, the script is currently only tested agains the [Mobile Device Management Protocol Reference](https://developer.apple.com/library/content/documentation/Miscellaneous/Reference/MobileDeviceManagementProtocolRef) page, so I would expect a few more tweaks are neccesseary until it's truly against Apples site design. But it's a good start.

I will probably updated the Profile reference page using this new script in the future.
