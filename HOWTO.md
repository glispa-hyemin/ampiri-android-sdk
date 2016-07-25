# Ampiri Android SDK 3.x Integration Guide

* [Updating your Android Manifest](#updating-your-android-manifest)
* [Standard banners](#markdown-header-standard-banners)
* [Interstitials](#markdown-header-interstitial-ads)
* [Video](#markdown-header-video-ads)
* [Native Ads](#markdown-header-native-ads)
* [In Feed Ads](#markdown-header-in-feed-ads)
* [Ad events handling](#markdown-header-ad-events-handling)
* [Activity lifecycle events handling](#markdown-header-activity-lifecycle-events-handling)
* [User Data](#markdown-header-user-data)
* [Log](#markdown-header-log)
* [Debug mode](#markdown-header-debug-mode)
* [Test devices](#markdown-header-test-devices)

## Updating your Android Manifest

Under the main `<manifest>` element, add the following permissions.

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
```

* ACCESS_COARSE_LOCATION (recommended) - Grants the SDK permission to access approximate location based on cell tower.
* ACCESS_FINE_LOCATION (recommended) - Grants the SDK permission to access a more accurate location based on GPS.

Although not technically required, the LOCATION permissions make it possible for the SDK to send location-based data to advertisers. Sending better
location data generally leads to better monetization.

* WRITE_EXTERNAL_STORAGE (optional) - Allows the SDK to cache all ad assets (creatives, custom frames, etc.) in external memory. This can maximize
performance by ensuring immediate delivery of ads and minimize network traffic used by the SDK by keeping cached
ad assets available even after the user closes the app.

* READ_PHONE_STATE (recommended) - Allows the SDK to handle calls interrupting video playback during videos.

> When using SDK as a library project, you shouldn't need to worry about merging AndroidManifest.xml changes or Proguard settings. If you run into problems,
make sure `manifestmerger.enabled` is set to `true` in `project.properties`

## Standard banners

> Note: All SDK method calls should be done from the main thread (Main thread, UI thread).

Add a banner to layout file, e.g.:
```xml
<FrameLayout
    android:id="@+id/ad_view"
    android:layout_width="320dp"
    android:layout_height="50dp"
    android:background="@android:color/white"/>
```

It is advisable to make the banner size in the layout the same as the required one (see below). Otherwise, the banner might be displayed incorrectly.

Add the following code to your activity:
```java
FrameLayout adView = (FrameLayout) view.findViewById(R.id.ad_view);
StandardAd standardAd = new StandardAd(this, adView, "YOUR_STANDARD_AD_PLACE_ID", BannerSize.BANNER_SIZE_320x50, adListener);
standardAd.loadAd();
```

Banners `320*50` are served by default. Available sizes:

* 320x50
* 728x90

### Standard banner auto-update

You can switch the banner auto-update function on or off; to do this, call `setAutorefreshEnabled()` method, e.g.:
```java
standardAd.setAutorefreshEnabled(false);
```

By default, auto-update is switched on. The auto-update period is set up via the admin panel.

## Interstitial Ads

> Note: All SDK method calls should be done from the main thread (Main thread, UI thread).

### Interstitial ad initialization

Add the following code to your activity:
```java
InterstitialAd interstitialAd = new InterstitialAd(this, "YOUR_INTERSTITIAL_AD_PLACE_ID", adListener);
interstitialAd.loadAd();
```
After calling the `loadAd()` method, the interstitial download starts. If you call `loadAd()` again before the banner is fully served, the previous request processing is cancelled. In this case, only the last request will be processed.

When the banner download is completed, you can display the banner by calling `showAd()` method.
```java
interstitialAd.showAd();
```

To learn about download completion, subscribe to banner events (see [Ad events handling](#markdown-header-ad-events-handling)) or call method `isReady()`.
```java
interstitialAd.isReady();
```

If your application workflow allows showing full screen banners at any time and in any place, there are 2 additional ways to show it right after the loading has finished or with a custom delay after method invocation.

To load and show full screen banner right after it was loaded use:
```java
interstitialAd.loadAndShow()
```

To load and show full screen banner with a custom delay after method invocation use:
```java
interstitialAd.loadAndShowWithDelay()
```
The delay interval is specified via Admin UI interface.

If you want full control over when and where to show full screen banners, use the following steps:

1. Call `interstitialAd.loadAd()` in advance
2. Set `AdEventCallback` to handle banner events
3. When you want to show the banner, check that it is ready and show: `if (interstitialAd.isReady()) interstitialAd.showAd()`
4. Start loading next banner in `onAdClosed()` event handler of `AdEventCallback`

## Video Ads

> Note: All SDK method calls should be done from the main thread (Main thread, UI thread).

### Video ad initialization

Add the following code to your activity:
```java
VideoAd videoAd = new VideoAd(this, "YOUR_VIDEO_AD_PLACE_ID", adListener);
videoAd.loadAd();
```

Close button for video ads is supported for some networks. If you want to enable this button you should add boolean parameter in video ad constructor:
```java
VideoAd videoAd = new VideoAd(this, "YOUR_VIDEO_AD_PLACE_ID", closeButtonEnabled);
or
VideoAd videoAd = new VideoAd(this, "YOUR_VIDEO_AD_PLACE_ID", closeButtonEnabled, adListener);
```

After calling the `loadAd()` method, the video download starts. If you call `loadAd()` again before the video has started to show, new request processing is cancelled. Only one request will be processed.

When the video download is completed, you can display it by calling the `showAd()` method.
```java
videoAd.showAd();
```

To learn about download completion, subscribe to video banner events (see [Ad events handling](#markdown-header-ad-events-handling) section) or call method `isReady()`.
```java
videoAd.isReady();
```

## Native Ads

>Note: All SDK method calls should be done from the main thread (Main thread, UI thread).

Native ads are loaded via the `NativeAd` class, which has its own `Builder` class to customize it during creation:

```java
NativeAd nativeAd = new NativeAd.Builder()
  .setAdPlaceId(YOUR_NATIVE_AD_PLACE_ID)
  .setCallback(adListener)
  .setAdAttributionText(getString(R.string.ad_attribution_text))
  .build(this);
```

To show native ads you can use two methods:

* Create an ad view programmatically from template and add it to the screen.
* Add `NativeAdView` view in the layout and bind loaded data to this view.

### Templates

Ampiri SDK provides 3 types of templates for native ads

* FeedCardNativeAdView - Icon, title, description, star rating, and CTA button
* StoryCardNativeAdView - Icon, image, title, description, star rating, and CTA button
* VideoCardNativeAdView - Icon, image/video/carousel, title, description, star rating, and CTA button

> Every template has a label that clearly indicates that it is an ad. For example "Ad" or "Sponsored".

If you want to use one of these templates, you can add the selected template in the creation of the `NativeAd`:

```java
.setAdViewBuilder(FeedCardNativeAdView.BUILDER);
```

With a native template, you can customize the following elements:

* Height
* Width
* Background Color
* Title Color
* Title Font
* Description Color
* Description Font
* Button Color
* Button Title Color
* Button Title Font
* Button Border Color

In order to customize these elements, you will need to build an attributes object and provide the following in the creation of the `NativeAd`:
```java
.setAdView(FeedCardNativeAdView.BUILDER, new NativeAdView.Attributes()
    .setBackgroundColor(Color.RED)
    .setTitleTextColor(Color.GREEN)
    .setButtonColor(Color.GREEN));
```

Add a banner place to layout, e.g.:
``` xml
  <FrameLayout
    android:id="@+id/ad_container"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:visibility="gone"/>
```

After calling the `loadAd()` method, the ad download starts. If you call `loadAd()` again before the banner is fully served, new request processing is
ignored. In this case, only the last request will be processed.

When the banner download is completed, you can display the banner by calling `renderAdView()` method.

```java
adContainerView = (FrameLayout) view.findViewById(R.id.ad_container);

@Override
public void onAdLoaded() {
  adContainerView.setVisibility(View.VISIBLE);
  adContainerView.removeAllViews();
  adContainerView.addView(nativeAd.renderAdView());
}
```

### Create native UI

In order to start using native ads, you will need to go through following steps:

* Create all needed views (icon view, main image view, text views, rating bar etc...)
* Pass the views to our SDK

You can either create your custom views in a layout `.xml`, or you can add elements in the code.

> All views should be placed in one child; this child itself should be placed in `NativeAdView`.

Custom layout `.xml`. For example:

``` xml
<com.ampiri.sdk.banner.NativeAdView android:id="@+id/native_ad"
 ...>
    <RelativeLayout ...>
        <ImageView android:id="@+id/native_ad_icon"
          ... />
        <ImageView android:id="@+id/native_ad_cover_image"
          ... />
        <FrameLayout android:id="@+id/native_ad_media_container"
          ... />
        <TextView android:id="@+id/native_ad_title"
          ... />
        <TextView android:id="@+id/native_ad_text"
          ... />
        <RatingBar android:id="@+id/native_ad_star_rating"
          ... />
        <Button android:id="@+id/native_ad_call_to_action"
          ... />
        <TextView android:id="@+id/native_ad_attribution"
          ... />
        <ImageView android:id="@+id/native_ad_choices_icon"
          android:layout_width="40dp"
          android:layout_height="40dp"
          android:padding="10dp"
          ... />
        <FrameLayout
          android:id="@+id/native_ad_choices_container"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:minHeight="20dp"
          android:minWidth="20dp"
          ... />
    </RelativeLayout>
</com.ampiri.sdk.banner.NativeAdView>
```

After you created all the views, please proceed by passing the views to our SDK. For example:

```java
  adView = (NativeAdView) view.findViewById(R.id.native_ad);

  adView.setIconView(R.id.native_ad_icon);
  adView.setCoverImageView(R.id.native_ad_cover_image);
  adView.setMediaContainerView(R.id.native_ad_media_container);
  adView.setTextView(R.id.native_ad_text);
  adView.setTitleView(R.id.native_ad_title);
  adView.setCallToActionView(R.id.native_ad_call_to_action);
  adView.setStarRatingView(R.id.native_ad_star_rating);
  adView.setAdAttributionView(R.id.native_ad_attribution);
  adView.setAdChoiceIconView(R.id.native_ad_choices_icon);
  adView.setAdChoiceContainerView(R.id.native_ad_choices_container);
```

Registering the native ad view in the creation of the `NativeAd`:

```java
.setAdView(adView);
```

After calling the `loadAd()` method, the ad download starts. If you call `loadAd()` again before the banner is fully served, new request processing is
ignored. In this case, only the last request will be processed.

When banner download is completed, you can display the banner by calling `showAd()` method.

To learn about download completion, subscribe to ad events (see [Ad events handling](#markdown-header-ad-events-handling) section) or call method `isReady()`.
```java
nativeAd.isReady();
```

## In Feed Ads

> Note: All SDK method calls should be done from the main thread (Main thread, UI thread).

Add the following code to your activity:

```java
StreamAdAdapter adAdapter = new StreamAdAdapter(this, new MainAdapter(this), "YOUR_NATIVE_AD_PLACE_ID", FeedCardNativeAdView.BUILDER, getString(R.string.ad_attribution_text));
listView.setAdapter(adAdapter);
adAdapter.loadAd();
```

After calling the `loadAd()` method, the in-feed ad download starts. If you call `loadAd()` again before the native ad is fully served, new request processing is cancelled. Only one request will be processed.

When in-feed ad download is completed, it will show automatically.

To learn about download completion, subscribe to ad events (see [Ad events handling](#markdown-header-ad-events-handling) section).


## Ad events handling

To receive events from ad, you should implement an event listener interface `AdEventCallback`.

Listener example:
```java
AdEventCallback adListener = new AdEventCallback() {
    @Override
    public void onAdLoaded() {
    }

    @Override
    public void onAdFailed(@NonNull final ResponseStatus responseStatus) {
    }

    @Override
    public void onAdOpened() {
    }

    @Override
    public void onAdClicked() {
    }

    @Override
    public void onAdClosed() {
    }

    @Override
    public void onAdCompleted() {
    }
};
```

## Activity lifecycle events handling

`onPause()`, `onResume()` and `onDestroy()` methods should be called depending on the activity lifecycle events.

Example:
```java
@Override
protected void onPause() {
    super.onPause();
    interstitialAd.onActivityPaused();
    standardAd.onActivityPaused();
    videoAd.onActivityPaused();
    nativeAd.onActivityPaused();
}

@Override
protected void onResume() {
    super.onResume();
    interstitialAd.onActivityResumed();
    standardAd.onActivityResumed();
    videoAd.onActivityResumed();
    nativeAd.onActivityResumed();
}

@Override
protected void onDestroy() {
    super.onDestroy();
    interstitialAd.onActivityDestroyed();
    standardAd.onActivityDestroyed();
    videoAd.onActivityDestroyed();
    nativeAd.onActivityDestroyed();
}
```

## User Data

To pass user data to the Ampiri SDK, use the following static methods:
```java
Ampiri.setUserBirthday(data);
Ampiri.setUserGender(UserData.Gender.FEMALE);
Ampiri.setUserInterests(Arrays.asList("football", "auto", "cats")); // Just for example. Please set real interests.
```

## Log

The default log level is INFO. From the adb shell, you can change the log level to DEBUG, VERBOSE etc. using this command:

```
setprop log.tag.Ampiri_SDK DEBUG
```

```
setprop log.tag.MRAID DEBUG
```

```
setprop log.tag.VAST DEBUG
```

## Debug mode

If you want to log debug information, please install `AmpiriLogger.setDebugMode(true)` (false by default), then you will see the logs under `Ampiri_SDK` tag.
It is recommended that this option should be used for integration test purposes.

## Test devices

### AdMob

```java
Ampiri.addMediationAdapter(new AdMobMediation.Builder()
    .addTestDevice("HASHED_ID")
    .build());
```

### Facebook

```java
Ampiri.addMediationAdapter(new FacebookMediation.Builder()
    .addTestDevice("HASHED_ID")
    .build());
```
