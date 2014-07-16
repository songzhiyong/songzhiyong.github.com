---
layout: post
title: "Android Application class being called twice"
date: 2014-07-11 17:50
comments: true
categories: android
---
[http://stackoverflow.com/questions/3947052/android-application-class-being-called-twice](http://stackoverflow.com/questions/3947052/android-application-class-being-called-twice)



Yes if you used android:process then you have it running in a separate process, so when the service starts a new process is started for it and thus a new Application object for that process needs to be created.

But there is a more fundamental problem -- it is just not right for an Application object to start one of its services. It is important that you don't confuse Application with how you may think of an "application" in another OS. The Application object does not drive the app. It is just a global of state for the app in that process. In fact, the Application object is completely superfluous -- you never need one to right an Android application. Generally I actually recommend that people don't use it. It is more likely to cause trouble than anything else.

Another way to put this: what really defines an application is its collection of activity, service, receiver, and provider tags. Those are what are "launched." All an Application is, is something that is created as part of initializing an application's process. It has no lifecycle of its own, it is just there to serve the other real components in the app.

So just ignore Application when designing your app; it will reduce confusion. (In its place, I prefer to use global singletons for such state.)

Also as a general rule, I recommend not using android:process. There are certainly some uses for it, but the vast majority of the time it is not needed and just makes an application use more RAM, less efficient, and harder to write (since you can't take advantage of globals in a single process). It should be obvious to you if you reach a place where there is actually a good reason to use android:process.