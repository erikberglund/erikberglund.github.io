---
title:  "Configuration Profile Reference Changelog"
date:   2018-04-25 20:00:00
description: Version controlled mirror of Apple's Configuration Profile Reference
---

Apple's [Configuration Profile Reference](https://developer.apple.com/library/content/featuredarticles/iPhoneConfigurationProfileRef/Introduction/Introduction.html) is the only vendor documentation for the native configuration profile payload keys available. But the [revision history](https://developer.apple.com/library/content/featuredarticles/iPhoneConfigurationProfileRef/RevHist_Index/RevisionHistory.html) gives no information on what was added, updated or removed.

To address this problem I've made a [project](https://github.com/erikberglund/Configuration-Profile-Reference) available on GitHub where I scrape the [Configuration Profile Reference](https://developer.apple.com/library/content/featuredarticles/iPhoneConfigurationProfileRef/Introduction/Introduction.html) and convert it to markdown. And through the magic of the [Internet Archive](http://web.archive.org) I was able to get most iterations of the site from late 2016 up until now.

# How should I use it?

If you just want to read the site in markdown you can browse the markdown folder [here](https://github.com/erikberglund/Configuration-Profile-Reference/tree/master/markdown). 

(The file [_All.md](https://github.com/erikberglund/Configuration-Profile-Reference/blob/master/markdown/_All.md) contains all payloads combined, just like the official site).

If you're interested in all changes between subsequent document versions of the site you can view each commit starting with **Document Version:** in the commit section [here](https://github.com/erikberglund/Configuration-Profile-Reference/commits/master)

If you want to find out when a particular payload type was updated, you go to it's markdown file, for example: [Loginwindow Payload](https://github.com/erikberglund/Configuration-Profile-Reference/blob/master/markdown/Loginwindow%20Payload.md). On that page, click the **History** button at the top of the page contents to see all document versions in which this payload was modified.

# Unofficial Release Notes

I plan on compiling some unofficial release notes for future updates to the site to make it easier to see what has changed insted of clicking around in the diffs. 

I will post them on this blog and on my [twitter](https://twitter.com/ekkrik).

This work was also an important step to help my other project [ProfileCreator](https://github.com/erikberglund/ProfileCreator) stay up to date.
