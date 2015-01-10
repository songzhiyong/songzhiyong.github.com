---
layout: post
title: "RemoteViews.apply"
date: 2014-08-18 21:17
comments: true
categories: android
---
###RemoteViews.apply
View  apply(Context context, ViewGroup parent)

Inflates the view hierarchy represented by this object and applies all of the actions.

###USAGE:
```java
...bla bla bla

Notification notification = sbn.getNotification();
RemoteViews remoteViews = notification.contentView;
View notiView = previewRemote.apply(context, null);
...blablabla

```