---
layout: post
categories:
- java
- android
title: Using Gradle to change your API's endpoint per build
---

Forgetting to change your Android application's API endpoint when publishing to the Play Store is something I did at least three times when building SmartChat. They were all caught quickly and it was only an alpha app, so it wasn't ever a big issue. However, this is something I would like to avoid.

I searched a bit and found that you could set BuildConfig variables per release type. This lets you never forget to change it again.

##### MyProject/build.gradle
{% highlight groovy %}
android {
  buildTypes {
    debug {
      buildConfigField "String", "API_URL", "\"http://192.168.1.2:5000/\""
    }

    release {
      buildConfigField "String", "API_URL", "\"http://example.com/\""
    }
  }
}
{% endhighlight %}

With this setup you can access your API endpoint via `BuildConfig.API_URL`. This will be set at build time and you won't accidentally forget to change it back when building for the Play Store.

I also found being able to set it different during development was handy so I ended up with this method in my API client.

{% highlight java %}
public String getRootURL() {
  if (BuildConfig.DEBUG) {
    // Change me to whatever you want, this will only matter in development
    return BuildConfig.API_URL;
  }
  return BuildConfig.API_URL;
}
{% endhighlight %}
