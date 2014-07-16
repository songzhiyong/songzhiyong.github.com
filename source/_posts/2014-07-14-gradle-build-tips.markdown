---
layout: post
title: "Gradle build Tips"
date: 2014-07-14 10:42
comments: true
categories: android
---

I'm lazy. Hopefully you are too. Here's two small conveniences for the Gradle build system you might find useful:

Decomposed Version Name and Code

Break up your version into its logical components and manage them separately. Stop wondering if you've bumped the version code correctly or updated the version name properly.
```
    def versionMajor = 3
    def versionMinor = 0
    def versionPatch = 0
    def versionBuild = 0 // bump for dogfood builds, public betas, etc.

    android {
      defaultConfig {
        versionCode versionMajor * 10000 + versionMinor * 1000 + versionPatch * 100 + versionBuild
        versionName "${versionMajor}.${versionMinor}.${versionPatch}"
      }
    }
```
If you have more than 10 patch or minor versions you have other problems. 10 builds? Unlikely, but we've got a whole int32 to work with here so why not...

A More Useful BuildConfig

Build config is FINALLY ours to add to. Leverage it for constant information that might otherwise require a Context to obtain.
```
    def gitSha() {
      return 'git rev-parse --short HEAD'.execute().text.trim()
    }

    def buildTime() {
      def df = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm'Z'")
      df.setTimeZone(TimeZone.getTimeZone("UTC"))
      return df.format(new Date())
    }

    android {
      defaultConfig {
        buildConfig """\
          public static final String GIT_SHA = "${gitSha()}";
          public static final String BUILD_TIME = "${buildTime()}";
        """
      }
    }
```
And if you think BuildConfig is currently under-utilized, you're right. Let's get more utility into it! Vote @ http://b.android.com/59302



Since Gadle 1.9  

```
 buildConfigField "String" , "GIT_SHA" ,"\"${gitSha()}\""
 buildConfigField "String","BUILD_TIME","\"${buildTime()}\""
```


