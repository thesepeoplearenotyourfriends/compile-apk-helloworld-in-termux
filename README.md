# compile-apk-helloworld-in-termux

Build a tiny Android APK from a shell in Termux without Gradle, Maven, or an IDE.

This project has been refactored for current Android compatibility:

- Targets **Android 15 / API 35** (`targetSdkVersion="35"`).
- Uses **API 24+** as the supported install floor so the APK can rely on modern whole-file APK signing.
- Declares `android:exported="true"` on the launcher activity, which is required for apps targeting Android 12+ when an activity has an intent filter.
- Builds with `d8` when available, falling back to `dx` only for older Termux setups.
- Signs with `apksigner` using APK Signature Scheme **v2** and **v3**, and attempts **v4** `.idsig` generation when the installed build-tools support it.
- Keeps signing secrets out of the script by allowing environment variable overrides.

## Install tools in Termux

```sh
pkg install aapt2 d8 zip openjdk-17 apksigner zipalign
```

Package names can vary by Termux repository vintage. The script will also use compatible tools placed in `project/toolz/`.

## Build

```sh
./build.sh project
```

The signed APK is written to:

```text
project/build/final.apk
```

If v4 signing is supported, an incremental-install signature may also be written to:

```text
project/build/final.apk.idsig
```

## Android platform jar

For best Android 15 compatibility, build with an API 35 platform jar. The script searches in this order:

1. `ANDROID_JAR` when explicitly set.
2. `$ANDROID_HOME/platforms/android-35/android.jar`.
3. `$ANDROID_SDK_ROOT/platforms/android-35/android.jar`.
4. `project/toolz/android.jar` as a local fallback.

Example:

```sh
ANDROID_JAR="$ANDROID_HOME/platforms/android-35/android.jar" ./build.sh project
```

## Signing

The repository includes a demo keystore so the sample can build immediately. Do **not** use that keystore for a real app.

For a real app, create your own keystore and pass credentials through the environment:

```sh
keytool -genkeypair \
  -keystore project/key.keystore \
  -alias mykey \
  -keyalg RSA \
  -keysize 4096 \
  -sigalg SHA256withRSA \
  -validity 10000

KEYSTORE_PASSWORD='your-password' KEY_ALIAS='mykey' ./build.sh project
```

The build script signs with:

- `--min-sdk-version 24`
- `--v1-signing-enabled false`
- `--v2-signing-enabled true`
- `--v3-signing-enabled true`
- `--v4-signing-enabled true` when supported by your `apksigner`

This favors modern APK integrity protections for Android 7.0+ devices. If you intentionally lower `minSdkVersion` below 24, revisit signing choices because Android 6.0 and older require legacy v1/JAR signing.

## Project layout

```text
build.sh                              Build, align, sign, and verify the APK.
project/AndroidManifest.xml           Android 15-compatible manifest.
project/res/layout/activity_main.xml  WebView layout.
project/res/values/values.xml         Strings and theme.
project/src/com/helloworld/           Java source.
project/toolz/                        Optional local build-tool fallbacks.
```
