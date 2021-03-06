# PureSDK for iOS

## Requirements
The iOS SDK is available for applications targeting iOS 9.0 and above.

## Installation
* This guide assumes you have obtained a key to use this
framework, please contact us if you don't yet have one. *

We support Cocoapods, Carthage, and Dynamic Framework installation.
If your app uses the `CoreBluetooth` framework, you may install the `PureSDKBluetooth` in addition to `PureSDK`. 
All of our frameworks have bitcode and simulator slices included.

After finishing installation, head to the `Usage` section to set the SDK up. The header file `Pure.h` contains documentation as well.
If you installed the `PureSDKBluetooth` framework, take care to read the `Bluetooth` section as well.

### Cocoapods

Use the following, with your key substituted in :

```ruby
use_frameworks!
pod 'PureSDK', :podspec => 'https://puresdk.azurewebsites.net/cocoapods/sdk/versions/latest?key=INSERT_KEY_HERE'
pod 'PureSDKBluetooth', :podspec => 'https://puresdk.azurewebsites.net/cocoapods/bluetooth/versions/latest?key=INSERT_KEY_HERE'
```

If you'd like to specify a certain version, you can use the following link(s) instead (without a .zip ending!) :
```
https://puresdk.azurewebsites.net/cocoapods/sdk/versions/1.0.95?key=INSERT_KEY_HERE
https://puresdk.azurewebsites.net/cocoapods/bluetooth/versions/1.0.95?key=INSERT_KEY_HERE
```

### Carthage

Use the following, with your key substituted in :

```ruby
binary "https://puresdk.azurewebsites.net/carthage/sdk/PureSDK.json?key=INSERT_KEY_HERE"
binary "https://puresdk.azurewebsites.net/carthage/sdk/PureSDKBluetooth.json?key=INSERT_KEY_HERE"
```

Make sure you've followed the instructions on the Carthage website, specifically :
1. Embedded Binaries should contain `Carthage/Build/iOS/PureSDK.framework` 
2. Run script calling `/usr/local/bin/carthage copy-frameworks` (and opt. input/output file setup)

### Dynamic Framework

Download the latest version of the framework(s) from :

`https://puresdk.azurewebsites.net/cocoapods/sdk/versions/1.0.95.zip?key=INSERT_KEY_HERE`
`https://puresdk.azurewebsites.net/cocoapods/bluetooth/versions/1.0.95.zip?key=INSERT_KEY_HERE`

1. Open your project in Xcode.
2. Drag and drop `PureSDK.framework` (and optionally `PureSDKBluetooth.framework`) into your project. Make sure the `Copy files` box is checked. *Uncheck* any selected targets. Click ok.
3. Select the target you wish to integrate the SDK into on the left-hand side of the project editor.
5. Find the `Embedded Binaries` section under `General`, and add the `PureSDK.framework` (and optionally `PureSDKBluetooth.framework`) that you just included into your project. This will also add the framework to the `Linked Libraries and Frameworks` section.
6. Navigate to `Build Phases`, click the small `+` in the left area of the window, and click `New Run Script Phase`.
7. Paste the following line(s) into the phase that was just added, and if you like rename the phase to "Strip Invalid Archs". This script (written by the excellent people at Realm!) removes architecture slices that are used on the iOS simulator.

```bash
bash "${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}/PureSDK.framework/strip-frameworks.sh"
bash "${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}/PureSDKBluetooth.framework/strip-frameworks.sh"
```

That's it! These instructions were last tested on Xcode 9.4 (9F1027a) and Xcode 10.1 (10B61).

For comparison, your `General` tab should look like this :

![finished-settings](https://github.com/unacast/pure-sdk-ios/blob/master/dynamic_framework_integration_result.png)

and the build phase "Strip Invalid Archs" should look like this :

![finished-run-script](https://github.com/unacast/pure-sdk-ios/blob/master/blt_run_script.png)

## Bluetooth

Apple requires all apps that import the `CoreBluetooth` framework to explicitly state its intention under the `NSBluetoothPeripheralUsageDescription` key in the application's `Info.plist`. Because of this, our bluetooth tracking code comes bundled as an optional framework (`PureSDKBluetooth.framework`).

To install the framework, first double check that the `NSBluetoothPeripheralUsageDescription` key is set to a reasonable string in your application's `Info.plist`. Failure to include this key will cause your app to be rejected on App Store Connect upload. Then, follow the instructions below depending on the preferred installation method. The `PureSDKBluetooth` framework cannot be installed independently from `PureSDK` since the bluetooth code requires the core SDK to function.

If you include the `bluetooth-central` background perimission, then we are able to provide much more detailed eddystone data for you.

Make sure you've followed the instructions on the Carthage website, specifically :
1. Embedded Binaries should contain `Carthage/Build/iOS/PureSDK.framework` and `Carthage/Build/iOS/PureSDKBluetooth.framework` 
2. Run script calling `/usr/local/bin/carthage copy-frameworks` (and opt. input/output file setup)

## Permissions

Some of the SDK's data collection sources require extra keys in `Info.plist`, and some require the user to accept a popup. We will never trigger the permission popups from our code. It is your responsibility to present the permission prompt alert at an appropriate time.

### Location

Here are the `Info.plist` keys that Apple requires for location tracking :

- `NSLocationWhenInUseUsageDescription` : Describes how your app will use location services in the foreground. You must include this key if you wish to collect location events.

- `NSLocationAlwaysAndWhenInUseUsageDescription` : Describes how your app will use location services and explains what extra features you can provide if the always option is selected. The presented alert gives the user the option between "Only when in use", "Always", and "Never". Only used on iOS 11 or later.

- `NSLocationAlwaysUsageDescription` : Describes how your app will use location services, both in the foreground and background. The presented alert gives the user the option between "Always", and "Never". If your deployment target is at iOS 11 or higher, then you don't need this key even if you wish to collect location events.

Recap : if your application is targeting iOS 11 or higher then only the `WhenInUseUsage` and `AlwaysAndWhenInUseUsage` plist keys are required. If your application is targeting iOS 10 or below, then you need to include all three of these keys.

Find your app's `Info.plist`, right click, select `Open As`, tap `Source Code` and paste the keys you will be using. Remember, these keys will be shown to your users.

```xml
<!-- All iOS versions -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>We'll show you interesting things around you.</string>

<!-- iOS 10 and below only -->
<key>NSLocationAlwaysUsageDescription</key>
<string>We'll show you interesting things around you.</string>

<!-- iOS 11 and above only -->
<key>NSLocationAlwaysAndWhenInUsageDescription</key>
<string>We'll show you interesting things around you, and with the "always" option, we'll also send you notifications when you come across something cool.</string>
```

## Usage

Two method calls are required to get the SDK to start tracking :

First in, `AppDelegate.m didFinishLaunchingWithOptions:`

```objc
[Pure initializeWithLaunchOptions:launchOptions];
```

Then, somewhere of your choosing (can be placed right after the above call if desired) : 

```objc
[Pure startTracking];
```

`initializeWithLaunchOptions` must be called before any other methods can be called on `Pure`

At any time, event collection can be stopped by calling :

```objc
[Pure stopTracking];
```

## Configuration

All config variables used by the SDK are provided by an external service. This means it's possible to change the behavior of the SDK without releasing a new version and configure settings per-device using filters based on device type, iOS version, etc.

## Custom Data

PureSDK provides additional methods to associate information with the current user session.

### Event

To create an "event", use :

```objc
[Pure createEventWithType:<#(nonnull NSString *)#> payload:<#(nonnull NSDictionary *)#> success:^{
    <#code#>
} failure:^(NSError * _Nullable error) {
    <#code#>
}];
```

The `type` describes what kind of event this is. Events are always added, not replaced.
Subsequent calls to `createEventWithType` with equal `type` values will create multiple events in the cloud. This method requires that the SDK has been started (`[Pure startTracking]`), and will return an error if that is not the case. To override this behavior, use the overloaded method and pass `YES` for the `force` parameter.

### Metadata

To associate some metadata with the current user, use :

```objc
[Pure associateMetadataWithType:<#(nonnull NSString *)#> payload:<#(nonnull NSDictionary *)#> success:^{
    <#code#>
} failure:^(NSError * _Nullable error) {
    <#code#>
}];
```

The `type` has to be unique for each payload you want to preserve. Subsequent calls to `associateMetadataWithType` with equal `type` values will overwrite the data sent on previous calls. This method requires that the SDK has been started (`[Pure startTracking]`), and will return an error if that is not the case. To override this behavior, use the overloaded method and pass `YES` for the `force` parameter.

## Event Types

Here is an overview of what kinds of data we collect and upload.

1. Location (always, whenInUse)
2. iBeacons (always, whenInUse)
3. Network Connection
4. Visits (always)

If the `PureSDKBluetooth` framework is installed, we also collect :

5. Eddystone UID
6. Eddystone URL

Sample Location Event :

```json
{
    "vacc":17.2101072,
    "updated":"2018-04-26T01:41:23+00:00",
    "hacc":1414.0000256,
    "lon":-122.1871488,
    "lat":37.8745792,
    "type":"LOCATION",
    "altitude":69.7735168,
    "timestamp":"2018-04-26T01:41:24+00:00"
}
```

Sample iBeacon Event :

```json
{
    "minor": 2,
    "rssi": 3,
    "major": 1,
    "type": "IBEACON",
    "timestamp": "2018-02-09T19:14:15-08:00",
    "uuid": "2F234454-CF6D-4A0F-ADF2-F4911BA9FFA6"
}
```

Each event has additional data sent with to help make our data collection more precise.
