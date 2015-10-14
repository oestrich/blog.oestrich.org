---
layout: post
categories:
- android
- google play services
- java
title: Using the New Google Play Service API
---

I recently got to use the new `FusedLocationApi` from Google Play Services on Android. This is something I have tried in the past, but all of the Android documentation refers to the deprecated `LocationClient`. I wanted to figure out how to use the new API so I did not need to update this project again.

```java
public class MainActivity extends Activity implements
  GoogleApiClient.ConnectionCallbacks,
  GoogleApiClient.OnConnectionFailedListener {

  private GoogleApiClient mGoogleApiClient;

  Button mLocationUpdates;

  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    mGoogleApiClient = new GoogleApiClient.Builder(this)
      .addApi(LocationServices.API)
      .addConnectionCallbacks(this)
      .addOnConnectionFailedListener(this)
      .build();

    mLocationUpdates = (Button) findViewById(R.id.location_updates);
    // Disable until connected to GoogleApiClient
    mLocationUpdates.setEnabled(false);
  }

  @Override
  public void onConnected(Bundle bundle) {
    mLocationUpdates.setEnabled(true);
    mLocationUpdates.setOnClickListener(new View.OnClickListener() {
      @Override
      public void onClick(View v) {
        locationRequest();
      }
    });
  }

  private void locationRequest() {
    LocationRequest request = new LocationRequest()
      .setInterval(1000) // Every 1 seconds
      .setExpirationDuration(60 * 1000) // Next 60 seconds
      .setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);

    Intent intent = new Intent(this, LocationUpdateService.class);
    PendingIntent pendingIntent = PendingIntent
      .getService(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
    LocationServices.FusedLocationApi
      .requestLocationUpdates(mGoogleApiClient, request, pendingIntent);
  }
}
```

This starts by creating a `GoogleApiClient` and setting connection callbacks to the activity. Once the `GoogleApiClient` is connected it calls `onConnected` and we enable the button that will start tracking location. I used the `IntentService` of receiving updates for this as it was simpler and worked for what I wanted. We create a `LocationRequest` whenever the button is pressed and ask for updates every second for 60 seconds.

```java
public class LocationUpdateService extends IntentService {
  public LocationUpdateService() {
    super("LocationUpdateService");
  }

  @Override
  protected void onHandleIntent(Intent intent) {
    Bundle bundle = intent.getExtras();

    if (bundle == null) {
      return;
    }

    Location location = bundle.getParcelable("com.google.android.location.LOCATION");

    if (location == null) {
      return;
    }

    // Deal with new location
  }
}
```

The `LocationUpdateService` is very simple and pulls the new updates out of the `Intent` it gets passed.

## Take Aways

I liked doing the `IntentService` route since we didn't have to worry about the activity being killed to keep receiving updates to location. It also got around the Android 5.1 "feature" of knocking all alarms to a 60 second interval no matter how small of a frequency you actually requested (to fetch `getLastLocation()` from the `LocationClient` as I had previously used.)
