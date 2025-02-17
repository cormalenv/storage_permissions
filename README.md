# Flutter Permission handler Plugin

[![pub package](https://img.shields.io/pub/v/storage_permissions.svg)](https://pub.dartlang.org/packages/storage_permissions)

A permissions plugin for Flutter. This plugin provides a cross-platform (iOS, Android) API to request and check permissions.

| Branch  | Build Status                                                                                                                                                       |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| develop | [![Build Status](https://app.bitrise.io/app/fa4f5d4bf452bcfb/status.svg?token=HorGpL_AOw2llYz39CjmdQ&branch=develop)](https://app.bitrise.io/app/fa4f5d4bf452bcfb) |
| master  | [![Build Status](https://app.bitrise.io/app/fa4f5d4bf452bcfb/status.svg?token=HorGpL_AOw2llYz39CjmdQ&branch=master)](https://app.bitrise.io/app/fa4f5d4bf452bcfb)  |

## NOTE

This is a fork from the original plugin [storage_permissions](https://github.com/Baseflow/flutter-permission-handler). To avoid the [issue](https://github.com/BaseflowIT/flutter-permission-handler/issues/26) with Apple Store asking you to include all permission options when you want to submit your App, I have removed all the permissions but the **storage** one.

So, use this plugin just to get storage permissions to your app.

## Features

- Check if a permission is granted.
- Request permission for a specific feature.
- Open app settings so the user can enable a permission.
- Show a rationale for requesting permission (Android).

## Usage

To use this plugin, add `storage_permissions` as a [dependency in your pubspec.yaml file](https://flutter.io/platform-plugins/). For example:

```yaml
dependencies:
  storage_permissions: "^4.0.0"
```

> **NOTE:** As of version 3.1.0 the storage_permissions plugin switched to the AndroidX version of the Android Support Libraries. This means you need to make sure your Android project is also upgraded to support AndroidX. Detailed instructions can be found [here](https://flutter.dev/docs/development/packages-and-plugins/androidx-compatibility).
>
> The TL;DR version is:
>
> 1.  Add the following to your "gradle.properties" file:
>
> ```
> android.useAndroidX=true
> android.enableJetifier=true
> ```
>
> 2.  Make sure you set the `compileSdkVersion` in your "android/app/build.gradle" file to 28:
>
> ```
> android {
>  compileSdkVersion 28
>
>  ...
> }
> ```
>
> 3.  Make sure you replace all the `android.` dependencies to their AndroidX counterparts (a full list can be found here: https://developer.android.com/jetpack/androidx/migrate).

### Android and iOS specific permissions

For this plugin to work you will have to add permission configuration to your `AndroidManifest.xml` (Android) and `Info.plist` (iOS) files. This will tell the platform which hardware or software features your app needs. Complete lists of these permission options can be found in our example app here:

- [AndroidManifest.xml](https://github.com/BaseflowIT/flutter-permission-handler/blob/develop/example/android/app/src/main/AndroidManifest.xml) (note that there is a debug, main and profile version which are used depending on how you start your App. In general it is sufficient to add permissions only to the `main` version);
- [Info.plist](https://github.com/BaseflowIT/flutter-permission-handler/blob/develop/example/ios/Runner/Info.plist)

> IMPORTANT: On iOS you will have to include all permission options when you want to submit your App. This is because the `storage_permissions` plugin touches all different SDKs and because the static code analyser (run by Apple upon App submission) detects this and will assert if it cannot find a matching permission option in the `Info.plist`. More information about this can be found [here](https://github.com/BaseflowIT/flutter-permission-handler/issues/26). Unfortunately we don't have a good solution for this.

## API

### Requesting permission

```dart
import 'package:storage_permissions/storage_permissions.dart';

Map<PermissionGroup, PermissionStatus> permissions = await PermissionHandler().requestPermissions([PermissionGroup.contacts]);
```

### Checking permission

```dart
import 'package:storage_permissions/storage_permissions.dart';

PermissionStatus permission = await PermissionHandler().checkPermissionStatus(PermissionGroup.contacts);
```

### Checking service status

```dart
import 'package:storage_permissions/storage_permissions.dart';

ServiceStatus serviceStatus = await PermissionHandler().checkServiceStatus(PermissionGroup.location);
```

Checking the service status only makes sense for the `PermissionGroup.location` on Android and the `PermissionGroup.location`, `PermissionGroup.locationWhenInUser`, `PermissionGroup.locationAlways` or `PermissionGroup.sensors` on iOS. All other permission groups are not backed by a separate service and will always return `ServiceStatus.notApplicable`.

### Open app settings

```dart
import 'package:storage_permissions/storage_permissions.dart';

bool isOpened = await PermissionHandler().openAppSettings();
```

### Show a rationale for requesting permission (Android only)

```dart
import 'package:storage_permissions/storage_permissions.dart';

bool isShown = await PermissionHandler().shouldShowRequestPermissionRationale(PermissionGroup.contacts);
```

This will always return `false` on iOS.

### List of available permissions

Defines the permission groups for which permissions can be checked or requested.

```dart
enum PermissionGroup {
  /// The unknown permission only used for return type, never requested
  unknown,

  /// Android: Calendar
  /// iOS: Calendar (Events)
  calendar,

  /// Android: Camera
  /// iOS: Photos (Camera Roll and Camera)
  camera,

  /// Android: Contacts
  /// iOS: AddressBook
  contacts,

  /// Android: Fine and Coarse Location
  /// iOS: CoreLocation (Always and WhenInUse)
  location,

  /// Android: Microphone
  /// iOS: Microphone
  microphone,

  /// Android: Phone
  /// iOS: Nothing
  phone,

  /// Android: Nothing
  /// iOS: Photos
  photos,

  /// Android: Nothing
  /// iOS: Reminders
  reminders,

  /// Android: Body Sensors
  /// iOS: CoreMotion
  sensors,

  /// Android: Sms
  /// iOS: Nothing
  sms,

  /// Android: External Storage
  /// iOS: Nothing
  storage,

  /// Android: Microphone
  /// iOS: Speech
  speech,

  /// Android: Fine and Coarse Location
  /// iOS: CoreLocation - Always
  locationAlways,

  /// Android: Fine and Coarse Location
  /// iOS: CoreLocation - WhenInUse
  locationWhenInUse,

  /// Android: None
  /// iOS: MPMediaLibrary
  mediaLibrary

  /// Android: Check notification enable
  /// iOS: Check and request notification permission
  notification
}
```

### Status of the permission

Defines the state of a permission group

```dart
enum PermissionStatus {
  /// Permission to access the requested feature is denied by the user.
  denied,

  /// Permissions to access the feature is granted by the user but the feature is disabled.
  disabled,

  /// Permission to access the requested feature is granted by the user.
  granted,

  /// The user granted restricted access to the requested feature (only on iOS).
  restricted,

  /// Permission is in an unknown state
  unknown
}
```

### Overview of possible service statuses

Defines the state of the backing service for the supplied permission group

```dart
/// Defines the state of a service related to the permission group
enum ServiceStatus {
  /// The unknown service status indicates the state of the service could not be determined.
  unknown,

  /// There is no service for the supplied permission group.
  notApplicable,

  /// The service for the supplied permission group is disabled.
  disabled,

  /// The service for the supplied permission group is enabled.
  enabled
}
```

## Issues

Please file any issues, bugs or feature request as an issue on our [GitHub](https://github.com/BaseflowIT/flutter-permission-handler/issues) page.

## Want to contribute

If you would like to contribute to the plugin (e.g. by improving the documentation, solving a bug or adding a cool new feature), please carefully review our [contribution guide](CONTRIBUTING.md) and send us your [pull request](https://github.com/BaseflowIT/flutter-permission-handler/pulls).

## Author

This Permission handler plugin for Flutter is developed by [Baseflow](https://baseflow.com). You can contact us at <hello@baseflow.com>
