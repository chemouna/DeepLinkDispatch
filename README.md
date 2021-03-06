# DeepLinkDispatch

[![Build Status](https://travis-ci.org/airbnb/DeepLinkDispatch.svg)](https://travis-ci.org/airbnb/DeepLinkDispatch)

DeepLinkDispatch provides a declarative, annotation-based API to declare application deep links.

You can register an `Activity` to handle specific deep links by annotating it with `@DeepLink` and a URI.
DeepLinkDispatch will parse the URI and dispatch the deep link to the appropriate `Activity`, along
with any parameters specified in the URI.

### Example

Here's an example where we register `SampleActivity` to pull out an ID from a deep link like
`example://example.com/deepLink/123`. We annotated with `@DeepLink` and specify there will be a
parameter that we'll identify with `id`.

```java
@DeepLink("foo://example.com/deepLink/{id}")
public class MainActivity extends Activity {
  @Override protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
  
    if (getIntent().getBooleanExtra(DeepLink.IS_DEEP_LINK, false)) {
      Bundle parameters = getIntent().getExtras();
    
      String idString = parameters.getString("id");
    
      // Do something with the ID...
    }
    ...
  }
}
```

### Multiple Deep Links

Sometimes you'll have an activity that should handle several kinds of deep links. You can use the
`@DeepLinks` annotation to register multiple deep links on an activity:

```java
@DeepLinks({"foo://example.com/deepLink/{id}", "foo://example.com/anotherDeepLink"})
public class MainActivity extends Activity {
  @Override protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
  
    if (getIntent().getBooleanExtra(DeepLink.IS_DEEP_LINK, false)) {
      Bundle parameters = getIntent().getExtras();
    
      String idString = parameters.getString("id");
    
      // Do something with the ID...
    }
    ...
  }
}
```

### Method Annotations

You can also annotate static methods that return an `Intent`. `DeepLinkDispatch` will call that
method to create that `Intent` and use it when starting your activity via that registered deep link:

```java
@DeepLink("foo://example.com/methodDeepLink/{param1}")
public static Intent intentForDeepLinkMethod(Context context) {
  return new Intent(context, MainActivity.class).setAction(ACTION_DEEP_LINK_METHOD);
}
```

### Query Parameters

Query parameters are parsed and passed along automatically, and are retrievable like any
other parameter. For example, we could retrieve the query parameter passed along in the URI
`example://example.com/deepLink?qp=123`:

```java
@DeepLink("foo://example.com/deepLink")
public class MainActivity extends Activity {
  @Override protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
  
    if (getIntent().getBooleanExtra(DeepLink.IS_DEEP_LINK, false)) {
      Bundle parameters = getIntent().getExtras();
    
      if (parameters != null && parameters.getString("qp") != null) {
        String queryParameter = parameters.getString("qp");
        
        // Do something with the query parameter...
      }
    }
  }
}
```

### Callbacks

You can optionally register callbacks to be called on any deep link success or failure.
`DeepLinkActivity` will broadcast an intent with any success or failure when deep linking. The intent
will be populated with these extras:

* `DeepLinkActivity.EXTRA_URI`: The URI of the deep link.
* `DeepLinkActivity.EXTRA_SUCCESSFUL`: Whether the deep link was fired successfully.
* `DeepLinkActivity.EXTRA_ERROR_MESSAGE`: If there was an error, the appropriate error message.

You can register a receiver to receive this intent. An example of such a use is below:

```java
public class DeepLinkReceiver extends BroadcastReceiver {

  private static final String TAG = DeepLinkReceiver.class.getSimpleName();

  @Override
  public void onReceive(Context context, Intent intent) {
    String deepLinkUri = intent.getStringExtra(DeepLinkActivity.EXTRA_URI);

    if (intent.getBooleanExtra(DeepLinkActivity.EXTRA_SUCCESSFUL, false)) {
      Log.i(TAG, "Success deep linking: " + deepLinkUri);
    } else {
      String errorMessage = intent.getStringExtra(DeepLinkActivity.EXTRA_ERROR_MESSAGE);
      Log.e(TAG, "Error deep linking: " + deepLinkUri + " with error message +" + errorMessage);
    }
  }
}
```

## Usage

Add to your project `build.gradle` file:

```groovy
buildscript {
  dependencies {
    classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
  }
}

apply plugin: 'com.neenbedankt.android-apt'

dependencies {
  compile 'com.airbnb:deeplinkdispatch:1.3.0'
  apt 'com.airbnb:deeplinkdispatch-processor:1.3.0'
}
```

Snapshots of the development version are available in [Sonatype's `snapshots` repository](https://oss.sonatype.org/content/repositories/snapshots/).

Register `DeepLinkActivity` with the scheme you'd like in your `AndroidManifest.xml` file (using
`airbnb` as an example):

```xml
<activity
    android:name="com.airbnb.deeplinkdispatch.DeepLinkActivity"
    android:theme="@android:style/Theme.NoDisplay">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="airbnb" />
    </intent-filter>
</activity>
```

That's it. The library will generate the class `DeepLinkActivity` during compilation.

## Testing the sample

Use adb to launch deep links.

This fires a standard deep link:

`adb shell am start -W -a android.intent.action.VIEW -d "airbnb://example.com/deepLink" com.airbnb.deeplinkdispatch.sample`

This fires a deep link associated with a method, and also passes along a parameter:

`adb shell am start -W -a android.intent.action.VIEW -d "airbnb://methodDeepLink/abc123" com.airbnb.deeplinkdispatch.sample`

## License

```
Copyright 2015 Airbnb, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
