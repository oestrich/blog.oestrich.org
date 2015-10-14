---
layout: post
categories:
- android
- gradle
title: Change Application Name per Release Type with Gradle
---

This is a handy techinique when used in conjuction with the previous week's tip with separate builds per release type.

To override the application name (and any other resource really), create a new folder under `app/src` that matches your release name. In this case we'll be overriding the debug release, so `app/src/debug`.

Once you have that simply create the same folder structure that is inside `app/src/main` for overrides. For overriding strings we'll create `app/src/debug/res/values`.

Inside the new `values` folder create `strings.xml` and add the new `app_name` value.

##### app/src/debug/res/values/strings.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

  <string name="app_name">MyApp Debug</string>

</resources>
```

Now when you install the new debug only application it will have a new name to distinguish between the various release types!
