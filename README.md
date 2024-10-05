# Flutter & Shorebird Release GitHub Action

This GitHub Action automates the build and release process for Flutter applications with **Shorebird** integration. It simplifies creating Flutter and Shorebird releases, applying Shorebird patches, and handling Android signing, all within your CI pipeline.

## Key Features

- **Flutter Release Builds**: Supports creating Android (APK, AAB) and iOS releases.
- **Shorebird Release**: Build and release Android and iOS applications with Shorebird.
- **Shorebird Patch**: Apply patches to both Android and iOS using Shorebird.
- **Android Signing**: Automatically sets up Android signing from GitHub secrets.
- **Output Versioning**: Outputs the release version for tracking.

## Inputs

### Flutter Build Inputs

- **`FLUTTER_VERSION`**: The Flutter version to use (e.g., `3.19.6`). Default: `3.19.6`.
- **`FLUTTER_EXTRA_ARGS`**: Additional arguments to pass to Flutter build commands. For example, `-v` for verbose output.
- **`FLUTTER_BUILD_APK`**: Build APK (Android Package). Set to `true` to enable. Default: `false`.
- **`FLUTTER_BUILD_AAB`**: Build Android App Bundle (AAB). Set to `true` to enable. Default: `false`.
- **`FLUTTER_BUILD_WEB`**: Build for the web. Set to `true` to enable. Default: `false`.

### Shorebird Release Inputs

- **`SHOREBIRD_RELEASE`**: Perform a Shorebird release. Set to `true` to enable. Default: `false`.
- **`SHOREBIRD_BUILD_IOS`**: Build Shorebird for iOS. Set to `true` to enable. Default: `false`.
- **`SHOREBIRD_BUILD_ANDROID`**: Build Shorebird for Android. Set to `true` to enable. Default: `false`.
- **`SHOREBIRD_EXTRA_ARGS`**: Additional arguments to pass to the Shorebird CLI. For example, `-v`.

### Shorebird Patch Inputs

- **`SHOREBIRD_PATCH`**: Apply a Shorebird patch. Set to `true` to enable. Default: `true`.
- **`SHOREBIRD_PATCH_RELEASE_VERSION`**: The release version to patch. Required if `SHOREBIRD_PATCH` is set to `true`. Default: `1.0.0+4`.
- **`SHOREBIRD_PATCH_ANDROID`**: Apply the Shorebird patch for Android. Set to `true` to enable. Default: `true`.
- **`SHOREBIRD_PATCH_IOS`**: Apply the Shorebird patch for iOS. Set to `true` to enable. Default: `false`.

### Shorebird Setup

- **`SHOREBIRD_TOKEN`**: Shorebird authentication token. Only required if building or patching using Shorebird.

### GitHub Android Signing Inputs

- **`ANDROID_KEYSTORE_BASE64`**: Base64-encoded Android keystore file.
- **`ANDROID_KEYSTORE_PASSWORD`**: Password for the Android keystore.
- **`ANDROID_KEY_ALIAS`**: Alias for the Android key.
- **`ANDROID_KEY_PASSWORD`**: Password for the Android key.

## How Android Signing Works

### Creating a New Keystore

To securely handle your Android signing credentials, the action uses a Base64-encoded keystore file. Here's how it works:

1. **Base64 Decoding**: The `ANDROID_KEYSTORE_BASE64` input should contain your Android keystore file encoded in Base64. During the workflow execution, the action decodes this string and saves it as a `keystore.jks` file in the runner's home directory.

    ```yaml
    - name: Decode Keystore
      if: ${{ inputs.ANDROID_KEYSTORE_BASE64 }}
      run: echo "${{ inputs.ANDROID_KEYSTORE_BASE64 }}" | base64 --decode > $HOME/keystore.jks
      shell: bash
    ```

2. **Updating `key.properties`**: After decoding the keystore, the action updates the `key.properties` file located in the `android` directory of your Flutter project. This file contains the necessary credentials for signing your Android builds.

    ```yaml
    - name: Update key.properties
      if: ${{ inputs.ANDROID_KEYSTORE_BASE64 }}
      run: |
        echo "storePassword=${{ inputs.ANDROID_KEYSTORE_PASSWORD }}" >> android/key.properties
        echo "keyPassword=${{ inputs.ANDROID_KEY_PASSWORD }}" >> android/key.properties
        echo "keyAlias=${{ inputs.ANDROID_KEY_ALIAS }}" >> android/key.properties
        echo "storeFile=$HOME/keystore.jks" >> android/key.properties
      shell: bash
    ```

### Overriding `key.properties`

The `key.properties` file is crucial for Android app signing. The action ensures that your signing credentials are correctly injected by appending the necessary properties:

- **`storePassword`**: Password for the keystore.
- **`keyPassword`**: Password for the specific key within the keystore.
- **`keyAlias`**: Alias for the key.
- **`storeFile`**: Path to the keystore file.

By dynamically updating `key.properties`, the action ensures that your Android builds are signed correctly without exposing sensitive information directly in your repository.

## Example Workflow Usage

```yaml
name: Build & Release

on:
  pull_request:
    branches:
      - stable
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Build & Release
        uses: crazidev/flutter-shorebird-action@v1
        with:
          FLUTTER_VERSION: "3.19.6"
          FLUTTER_BUILD_APK: false
          FLUTTER_BUILD_AAB: true
          FLUTTER_BUILD_WEB: false

          SHOREBIRD_RELEASE: true
          SHOREBIRD_BUILD_IOS: false
          SHOREBIRD_BUILD_ANDROID: true

          SHOREBIRD_PATCH: true
          SHOREBIRD_PATCH_RELEASE_VERSION: "1.0.0+1"
          SHOREBIRD_PATCH_ANDROID: true
          SHOREBIRD_PATCH_IOS: false
          SHOREBIRD_TOKEN: ${{ secrets.SHOREBIRD_TOKEN }}

          ANDROID_KEYSTORE_BASE64: ${{ secrets.ANDROID_KEYSTORE_BASE64 }}
          ANDROID_KEYSTORE_PASSWORD: ${{ secrets.ANDROID_KEYSTORE_PASSWORD }}
          ANDROID_KEY_ALIAS: ${{ secrets.ANDROID_KEY_ALIAS }}
          ANDROID_KEY_PASSWORD: ${{ secrets.ANDROID_KEY_PASSWORD }}
```

## Inputs Overview

Below is a quick reference to the action's inputs:

| Input                           | Description                                         | Required | Default         |
|---------------------------------|-----------------------------------------------------|----------|-----------------|
| `FLUTTER_VERSION`               | Flutter version to use.                             | true     | `3.19.6`        |
| `FLUTTER_EXTRA_ARGS`            | Additional arguments for Flutter build commands.    | false    | `""`            |
| `FLUTTER_BUILD_APK`             | Build APK.                                          | false    | `false`         |
| `FLUTTER_BUILD_AAB`             | Build AAB.                                          | false    | `false`         |
| `FLUTTER_BUILD_WEB`             | Build for web.                                      | false    | `false`         |
| `SHOREBIRD_RELEASE`             | Perform a Shorebird release.                        | false    | `false`         |
| `SHOREBIRD_BUILD_IOS`           | Build Shorebird for iOS.                            | false    | `false`         |
| `SHOREBIRD_BUILD_ANDROID`       | Build Shorebird for Android.                        | false    | `false`         |
| `SHOREBIRD_EXTRA_ARGS`          | Additional arguments for Shorebird CLI.             | false    | `""`            |
| `SHOREBIRD_PATCH`               | Apply Shorebird patch.                              | false    | `true`          |
| `SHOREBIRD_PATCH_RELEASE_VERSION` | Shorebird patch release version.                   | false    | `1.0.0+4`       |
| `SHOREBIRD_PATCH_ANDROID`       | Apply Shorebird patch for Android.                  | false    | `true`          |
| `SHOREBIRD_PATCH_IOS`           | Apply Shorebird patch for iOS.                      | false    | `false`         |
| `SHOREBIRD_TOKEN`               | Shorebird authentication token.                     | false    | `""`            |
| `ANDROID_KEYSTORE_BASE64`       | Base64-encoded Android keystore file.               | false    | `""`            |
| `ANDROID_KEYSTORE_PASSWORD`     | Password for the Android keystore.                  | false    | `""`            |
| `ANDROID_KEY_ALIAS`             | Alias for the Android key.                          | false    | `""`            |
| `ANDROID_KEY_PASSWORD`          | Password for the Android key.                       | false    | `""`            |

## Setup and Configuration

### Shorebird Token

Ensure you add your **Shorebird Token** to your repository secrets as `SHOREBIRD_TOKEN`. This token is required if you are performing a Shorebird release or applying a patch.

### Android Signing

For Android signing, store the following values as secrets in your repository:

- **`ANDROID_KEYSTORE_BASE64`**: Your Android keystore file encoded in Base64.
- **`ANDROID_KEYSTORE_PASSWORD`**: Password for the Android keystore.
- **`ANDROID_KEY_ALIAS`**: Alias for the Android key.
- **`ANDROID_KEY_PASSWORD`**: Password for the Android key.

These values are used to sign your Android release builds during the build process. The action decodes the keystore, creates the `keystore.jks` file, and updates the `key.properties` file with the provided credentials to ensure your builds are properly signed.
