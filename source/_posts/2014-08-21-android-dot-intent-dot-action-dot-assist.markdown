---
layout: post
title: "android.intent.action.ASSIST"
date: 2014-08-21 17:28
comments: true
categories: android
---
###public static final String ACTION_ASSIST
####Added in API level 16
Activity Action: Perform assist action.

Input: EXTRA_ASSIST_PACKAGE, EXTRA_ASSIST_CONTEXT, can provide additional optional contextual information about where the user was when they requested the assist. Output: nothing.

Constant Value: "android.intent.action.ASSIST"

###Register:
```xml
<intent-filter>
    <action android:name="android.intent.action.ASSIST" />

    <category android:name="android.intent.category.DEFAULT" >
    </category>
</intent-filter>
```
