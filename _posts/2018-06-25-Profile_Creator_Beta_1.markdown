---
title:  "Profile Creator (Beta 1)"
date:   2018-06-28 08:00:00
description: First beta of Profile Creator
---

In 2015 I started a project to make it easier to create configuration profiles. The current solutions were (and still are) poor and missing a lot of basic features.

I made a POC that did exactly what I needed and continued to write the app in Objective-C. Unfortunately, personal reasons caused me to drop everything until January this year when I picked up where I left off. In my time away, Swift had risen a lot among Apple developers, so to make this application better suited for the future (and open the door to more admins who wanted to contribute) I decided to rewrite the application in Swift.

Now the application is ready for a first Beta.

Download it here: [Profile Creator Releases](https://github.com/erikberglund/ProfileCreator/releases)

# Features

The primary goal of the beta is testing stability and the core of the application: creating, editing and exporting profiles.

In this version, you can only add keys and payloads from the pre-defined payload manifests. I've included most macOS payloads, two preference domains and three well-known Open Source applications whose preferences are preferred to be delivered as profiles:

* Munki
* NoMad
* NoMad Login  

On export, you can sign your profile with a signing certificate from the login keychain.

# Planned Features

There are many features I would like to add, here is a small subset in no particular order:

* Import existing profiles
* Show configuration errors
* Merge/Split profiles
* Custom payloads
* Custom keys in any payload
* Read preferences on disk and save as profile (like [mcxToProfile](https://github.com/timsutton/mcxToProfile) but with a GUI).
* Allow a custom path for the main window groups (where the profile setting files are saved).
* Read/Update/Add/Sync profiles to an MDM using their APIs.

# Feedback

If you decide to try the application and have feedback (it can be anything from language, the user interface to code). Please open an issue on the project GitHub: [Profile Creator Issues](https://github.com/erikberglund/ProfileCreator/issues).

You can also join the [#profilecreator](https://macadmins.slack.com/messages/C2N7MED7G) channel on the [MacAdmins Slack](https://macadmins.herokuapp.com) to ask questions or discuss a feature.

