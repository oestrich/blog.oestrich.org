---
layout: default
categroies:
- Android
title: Android ImageView
---

# [{{ page.title}}]({{ page.url }})
<span>Posted on {{ page.date | date_to_string }}</span>

Android's ImageView does a strange thing when you have it stretch to fit the width of the view but not the height. It creates a big square for the view instead of conforming to the image size.

This:

<script type="text/javascript" src="https://gist.github.com/1379095.js?file=layout_image_stretched.xml"></script>

Gives you:

![Android ImageView no bounds](/images/android-imageview-1.png)

In order to fix this you can set the property "android:adjustViewBounds" to true, and the view will conform to the size of the image.

This:

<script type="text/javascript" src="https://gist.github.com/1379095.js?file=layout_image.xml"></script>

Gives you:

![Android ImageView with bounds](/images/android-imageview-2.png)

Much better.

Found from: [http://stackoverflow.com/questions/5355130/android-imageview-size-not-scaling-with-source-image](http://stackoverflow.com/questions/5355130/android-imageview-size-not-scaling-with-source-image)
