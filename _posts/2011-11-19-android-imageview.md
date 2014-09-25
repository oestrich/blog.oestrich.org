---
layout: post
categroies:
- Android
title: Android ImageView
---

Android's ImageView does a strange thing when you have it stretch to fit the width of the view but not the height. It creates a big square for the view instead of conforming to the image size.

This:

{% highlight java %}
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <ImageView 
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:src="@drawable/red" />
</RelativeLayout>
{% endhighlight %}

Gives you:

![Android ImageView no bounds](/images/android-imageview-1.png)

In order to fix this you can set the property "android:adjustViewBounds" to true, and the view will conform to the size of the image.

This:

{% highlight java %}
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <ImageView 
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:src="@drawable/red" 
        android:adjustViewBounds="true"/>
</RelativeLayout>
{% endhighlight %}

Gives you:

![Android ImageView with bounds](/images/android-imageview-2.png)

Much better.

Found from: [http://stackoverflow.com/questions/5355130/android-imageview-size-not-scaling-with-source-image](http://stackoverflow.com/questions/5355130/android-imageview-size-not-scaling-with-source-image)
