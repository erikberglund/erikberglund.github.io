---
title:  "Profile Creator: Open Source"
date:   2019-07-16 09:00:00
description: New horizons.
---

## New Beginnings

I started working on ProfileCreator many years ago in Objective-C. Since then me and my family have gone through (and are still living in) many changes that affect my ability to work on spare time projects like this. This is one of the reasons I sometimes have taken months/years long breaks while working on this project.

We have moved back to our hometown and I have started a new job where I no longer am involved in the administration of Macs, which was the only opportunity I had for a job if we were moving here. This move has allowed us to finally start recovering and working to improve the everyday life of our family, so I'm very glad I am here now.

Unfortunately this means I no longer have a job related reason to keep up with the Mac admin community, and I also have less free time outside work. ProfileCreator has been a passion project of mine which is why I have kept working on it even after we moved here, but now the time has come where I need to stop working on things that don't directly contribute to our family, either economically or by doing things here at home.

So that is why I'm very sad to say I cannot continue to develop ProfileCreator.

## Open Source

The source for the project was removed after one of the betas and I never put it up after that. And to be completely honest, the biggest reason for that was the bad code quality. This was the project where I taught myself Swift. I have not read any Swift books or done any courses, which is why a bunch of the code in ths project is just bad. I have worked through some parts to bring them more inline with how I would like the code to be, but still, most of the core is still both ugly and overly complicated.

I had a dream of fixing that before opening the project up, but I no longer have time for that so I will have to upload it as it is.

So from this version, the current code will be available in the project repo on GitHub under the GNU General Public License v3.0.

## ProfilePayloads

Along with ProfileCreator there is a framework called [ProfilePayloads](https://github.com/ProfileCreator/ProfilePayloads) which is responsible for reading the manifests from the [PayloadManifests](https://github.com/ProfileCreator/ProfileManifests) repo. I started a great rewrite and improvement of that framework to bring it up to modern swift, but also to include more structural parts from the main ProfileCreator application that would make the app much more simple to understand. I never got time to finish that rewrite, but it contains a lot of great things, especially the parts to read installed profiles from a system. So check that out if you are interested in profiles: [ProfileKit](https://github.com/ProfileCreator/ProfileKit).

## Future

I will still be in the MacAdmins slack for a few months longer, answering questions about the source. If you want to continue developing this application, join the #profilecreator-dev channel and talk to me and the other persons in that channel to get help and information. I will be there answering any questions for anyone who wants to try to continue developing this app.

There is such a long list of features I wanted to get in, and so many internal changes I wanted to make so I cannot list the here. But a very short list is:

##### Changes I would have worked on next

- Download/Upload/Modify profiles in MDMs using the MDM APIs.
- Rewrite the main editing view using elements not available when I started. i.e make it dynamically change size etc.
- Rewrite the save structure. Currently the "isSelected" state for a single key is stored along with the settings. If that was broken out to a separate array, and the settings would be just like a finished profile, everything would be so much better in the app.
- Create a new UI for editing deeply nested keys.

    
## Finally

The hope for this application from the beginning was to actually create a proof of concept to show MDM vendors that MacAdmins really need much better granularity when creating profiles. And my goal has not been this app itself, the app was just a way to get by when the existing tools did such a poor job to help an admin do the work they needed. I felt like the developers of the tools did not have a grip with what the users of their MDM actually needed so I wanted to make this to fix things for myself, but also to show that there is at least one other way to solve this.

Now I heard that Jamfs profile rewrite is finally coming along. I have not seen it, but my hopes are that their changes will be what the community needs, and that this will spark the other vendors to also improve their UIs.

Thank you all for this time, it has been great. But now I have to move on to try and find other things to do that can help me and my family.

Good Bye 

/Erik
