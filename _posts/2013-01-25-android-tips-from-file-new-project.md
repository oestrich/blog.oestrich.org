---
layout: post
categories:
- android
title: Android Tips from File->New Project
---

I learned several Android tips the other day from an unlikely place: `File->New Project`. I started a new project and found out that the latest SDK gives you an option to include an activity as your base. I chose LoginActivity and found several new things.

### EditTexts can have errors

![Android EditText Error](/images/android-edit-text-error.png)

This is possible by calling `setError` on the EditText.

##### LoginActivity.java - attemptLogin()
{% highlight java %}
// Check for a valid password.
if (TextUtils.isEmpty(mPassword)) {
  mPasswordView.setError(getString(R.string.error_field_required));
  focusView = mPasswordView;
  cancel = true;
} else if (mPassword.length() < 4) {
  mPasswordView.setError(getString(R.string.error_invalid_password));
  focusView = mPasswordView;
  cancel = true;
}
{% endhighlight %}

### The merge view

I'm still a little uncertain about this one, but after doing some searching it looks like you can speed up the display of your views if you use `merge`. [More info](http://android-developers.blogspot.com/2009/03/android-layout-tricks-3-optimize-by.html).

##### activity_login.xml
{% highlight xml %}
<merge xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  tools:context=".LoginActivity" >

  <!-- Login progress -->

  <LinearLayout ... />
  <ScrollView ... />
</merge>
{% endhighlight %}

### Value files can be layout specific

This one might be my favorite tip of them. I knew you could make resource files specific to versions of android, but I didn't know you could make value files specific to a layout. With this you can stick layout specific strings in a separate file and not clutter up the main `strings.xml` file.

![Android values and layouts folder](/images/android-value-layout.png)
