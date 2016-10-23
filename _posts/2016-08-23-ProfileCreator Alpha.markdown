---
title:  "ProfileCreator: Alpha"
date:   2016-08-23 20:00:00
description: A first look and feel at ProfileCreator
---

_This is not going to be a technical post. I will tell a short history of why I'm creating this application, where I'm currently at and where I'm heading._

One year ago while converting an old workflow of scripted settings into configuration profiles I became frustrated that none of the tools I tried gave me enough granularity when choosing what to manage.

All tools seemed to copy the [Profile Manager](http://www.apple.com/osx/server/features/#profile-manager) GUI with a set of predetermined payloads that gathered a logical group of payload keys into a single setting. No tool offered the option to choose which of the predetermined keys in the payload I wanted. 

_I either had to manage all of them, or none of them._

This meant I had to do my configuration in the UI, export the configuration profile and manually remove the things I didn't want to force upon the user.

This process made me decide I had to create my own tool where I could choose exactly which keys I wanted to manage. And much more.

I named it [ProfileCreator](https://github.com/erikberglund/ProfileCreator).

![pfc_welcome.png]({{ site.url }}/assets/screenshots/pfc_welcome.png)

# Alpha

I created a proof of concept application to test the features I would like to see, and those tests have been very successful. But to make the application more structured and easier to maintain I started over with an empty project and have been rebuilding it for the past 2-3 months.

I have been asked about the progress of this project by a lot of people so I wanted to get it in the hands of admins as to test and leave feedback as soon as the foundation was ready.

Therefor, this first alpha release will have the following limitations:

* Only one payload type is available: **Certificates** (as per request of [Graham Gilbert](https://grahamgilbert.com))
* Only one payload per payload type (even if the payload type allows for more).
* No preview when selecting a profile in the main window.
* No indication if the payload contains an invalid configuration.
* No option to customize what payload keys are included.

Basically, you can only create a configuration profile containg a single certificate.

# Planned Features

I will continue to work hard on this project and implement all features I tested in my proof of concept as well as the features requested by the community. 

Here are some of the upcoming features:

 * Support for all Apple profile payloads.
 * Support for all Apple MCX settings.
 * Support for adding multiple payloads in a profile where the payload type allows it.
 * Ability to disable/enable single keys in any payload
 * Additive mode where you start with an empty payload and only add the keys you want.
 * Reading/Updating profiles on a JSS, or any mdm that offers an API for that.
 * Ability to create profiles from arbitrary preferences or plist files. (similar to [mcxToProfile](https://github.com/timsutton/mcxToProfile)).
 * Crowdsourced 3rd party preference profiles and their possible preference keys.
 * And many more...

Please add your feature requests or feedback on the [ProfileCreator GitHub](https://github.com/erikberglund/ProfileCreator/issues).

# Download

To try the application, download it from GitHub: [ProfileCreator: Releases](https://github.com/erikberglund/ProfileCreator/releases)

You can also chat with me and other mac admins about this project on the [MacAdmins Slack](https://macadmins.herokuapp.com) in the channel **[#profilecreator](https://macadmins.slack.com/archives/profilecreator)**.
