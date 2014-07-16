---
layout: post
title: "Dexguard Encrypt Asset File"
date: 2014-07-15 14:54
comments: true
categories: android
---

-encryptassetfiles [file_filter]

Specifies the Android asset files that should be encrypted. Asset files are stored in the assets directory and can contain any data. The obfuscation step can automatically encrypt them and make sure they are decrypted on the fly at run-time. In order for this to work, the asset files must be streamed and their names must be hard-coded. This means that your code must invoke the AsssetManager as follows: open("MyAssetFile"). Your configuration can then specify "-encryptassetfiles assets/MyAssetFile". Only applicable when obfuscating Android code.
