---
layout: post
categories:
- java
- android
- gradle
title: Using gradle to generate a separate development APK
---

Being able to have the same application installed for both development and the release version is very handy. It lets you always have a stable version of the app installed.

##### app/build.gradle

```groovy
android {
  // ...

  buildTypes {
    debug {
      // ...
      applicationIdSuffix ".dev"
    }
  }

  // ...
```
