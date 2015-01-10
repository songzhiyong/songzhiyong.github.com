---
layout: post
title: "Run lint in Android Studio"
date: 2014-09-19 10:52
comments: true
categories: android
---
Run lint in Android Studio

As I have just started learning Android developement, I was happy to see lint as such an integral part of the development process.

To runt lint and analyze your project, simply select `Analyze > Inspect Code`.

In the popup window that appears, you can select the analysis scope, like which projects to analyze and which to skip. For my test project, I just chose to analyze everything and ended up with this nice summary:
<br/><br/>
![](/images/2014/09/inspect_code.png "inspect code")<br/><br/>

When you browse through the reported items, you will notice that many of them can just be ignored. For instance, spelling, which catches a lot of non-typos in strings. For instance, the project name, is reported as a typo. It also complains about invalid XML in generated files.

However, stuff like the items under Android lint and Declaration redundancy are highly interesting, and will be a great help for a newbie like me.