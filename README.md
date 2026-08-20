# Dark Tricks - ColorOS Adaptation

Dark Tricks is an Xposed module that provides a collection of Android system
customizations. This fork is based on
[`darkeyes84/dark_tricks`](https://github.com/darkeyes84/dark_tricks) and adds
support for controlling the text cursor with the physical volume keys on
ColorOS.

## ColorOS cursor control

The upstream implementation handles volume keys inside `InputMethodService`.
On ColorOS 16, the system consumes those keys in `system_server` before Gboard
can receive them, so the original hook does not run.

This fork adds a system-level implementation that:

- Hooks `PhoneWindowManager.interceptKeyBeforeQueueing` before ColorOS sends
  volume events to the media-session stack.
- Tracks the real IME window visibility through `InputMethodManagerService`.
- Converts volume-key events to DPAD left/right events while the keyboard is
  visible.
- Suppresses the volume overlay and prevents media-volume changes while cursor
  control is active.
- Leaves the volume keys unchanged when the screen is off, the device is
  locked, the keyboard is hidden, or a call is active.
- Uses a short grace period to preserve power + volume and dual-volume-key
  combinations.
- Reloads module preferences after the Android user is unlocked.

With the default direction selected:

| Physical key | Result while the IME is visible |
| --- | --- |
| Volume up | Move the cursor left |
| Volume down | Move the cursor right |

The direction can be reversed in the module settings.

## Tested configuration

- OnePlus `PLR110`
- ColorOS 16 / Android 16
- APatch root
- Vector 2.2 (version code 3080, API 102)
- Gboard (`com.google.android.inputmethod.latin`)
- Microsoft Edge address bar and regular web text fields
- Dark Tricks `3.4-coloros3` (`versionCode 46`)

The cursor was verified through both Android key injection and the phone's raw
physical volume-key input device. While the IME was visible, the cursor moved,
the media volume remained unchanged, and no volume overlay appeared. Normal
volume control resumed after the IME was hidden.

## Requirements

- Android 14 or newer (`minSdk 34`)
- A rooted device with an Xposed-compatible framework such as Vector or LSPosed
- Xposed API 100 or newer for the module-loading metadata

This module hooks private Android framework APIs. Compatibility with other
devices, Android versions, and OEM ROMs is not guaranteed.

## Installation

1. Build or install the APK.
2. Enable Dark Tricks in Vector or another compatible Xposed manager.
3. Add **System Framework** (`system`/`android`) to the module scope. This scope
   is required for the ColorOS cursor-control implementation.
4. Add Gboard to the scope if the legacy IME fallback is also desired.
5. Add System UI, Outlook, Google Phone, or other packages only when using the
   corresponding original Dark Tricks features.
6. Open Dark Tricks and set **Volume keys control cursor** to the desired
   direction.
7. Reboot the device after enabling the module or changing its framework scope.

The foreground application, such as Edge, does not need to be added to the
scope for cursor control. The system hook injects the cursor event into the
currently focused text field.

## Other Dark Tricks features

The original project also contains options for:

- Status-bar and Quick Settings indicators
- Hiding VPN, certificate, ADB, alarm, and build-version notices
- Double-tap-to-sleep and quick-pulldown gestures
- Back-gesture height
- Proximity checks before wake-up
- Power-button torch control
- Screen-off volume-key track skipping
- Notification throttling
- Custom carrier text
- Outlook device-policy bypass
- Google Phone recording-warning suppression

Many of these hooks target specific Android framework or application versions
and have not been validated as part of the ColorOS cursor-control work.

## Building

The project uses Gradle 8.5 and Android Gradle Plugin 8.2.2.

Prerequisites:

- JDK 21
- Android SDK Platform 35
- Android SDK Build Tools 35.0.1

Build and run lint on Windows:

```powershell
$env:JAVA_HOME = 'C:\Program Files\Android\openjdk\jdk-21.0.8'
.\gradlew.bat assembleRelease lintRelease
```

Build on Linux or macOS:

```bash
JAVA_HOME=/path/to/jdk-21 ./gradlew assembleRelease lintRelease
```

The APK is generated at:

```text
app/build/outputs/apk/release/app-release.apk
```

To sign release builds with your own key, create an ignored
`keystore.properties` file in the repository root:

```properties
storeFile=/absolute/path/to/your.keystore
storePassword=your-store-password
keyAlias=your-key-alias
keyPassword=your-key-password
```

When `keystore.properties` is absent, the current build configuration falls
back to the Android debug signing key. Keep the same signing key for future
updates or Android will reject the APK as an incompatible update.

## Known limitations

- Framework internals can change after an Android or ColorOS update and may
  require new hooks.
- Preservation of screenshot and accessibility key combinations uses a
  180-millisecond chord window. Verify those combinations on each target ROM.
- The current compatibility work was tested with Gboard. Other IMEs may handle
  injected DPAD events differently.

## Upstream and licensing

Original project: [`darkeyes84/dark_tricks`](https://github.com/darkeyes84/dark_tricks)

The upstream repository does not currently contain a license file. Publicly
visible source code does not by itself grant redistribution rights. Contact the
upstream author before distributing modified source or APK releases outside
the permissions provided by the hosting platform.
