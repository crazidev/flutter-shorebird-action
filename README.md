# Flutter & Shorebird Release

This GitHub Action automates the build and release process for Flutter applications with Shorebird integration. It supports creating Flutter releases and Shorebird releases, as well as applying Shorebird patches for both Android and iOS platforms.

## Features

✅ Create Flutter Android/IOS releases

✅ Create Shorebird Android/IOS releases

✅ Create Shorebird Android/IOS patch

✅ Outputs the release version

## Inputs

### Flutter Build Inputs

- `FLUTTER_VESION`: Version of Flutter to use. Default: `3.19.6`.
- `FLUTTER_BUILD_APK`: Set to true to build APK. Default: `false`.
- `FLUTTER_BUILD_AAB`: Set to true to build AAB. Default: `false`.

### Shorebird Release Inputs

- `SHOREBIRD_RELEASE`: Set to true to perform a Shorebird release. Default: `false`.
- `SHOREBIRD_BUILD_IOS`: Set to true to build Shorebird for iOS. Default: `false`.
- `SHOREBIRD_BUILD_ANDROID`: Set to true to build Shorebird for Android. Default: `false`.

### Shorebird Patch Inputs

- `SHOREBIRD_PATCH`: Set to true to apply Shorebird patch. Default: `true`.
- `SHOREBIRD_PATCH_RELEASE_VERSION`: Shorebird patch release version. Required if `SHOREBIRD_PATCH` is true. Default: `1.0.0+4`.
- `SHOREBIRD_PATCH_ANDROID`: Set to true to apply Shorebird patch for Android. Default: `true`.
- `SHOREBIRD_PATCH_IOS`: Set to true to apply Shorebird patch for iOS. Default: `false`.

### Tokens

- `SHOREBIRD_TOKEN`: Shorebird token. Required.

### Android Signing

- `ANDROID_SIGNING_PRIVATE_PEM` Set a private key to be used for signing instead of using keystore/keypair, for security reasons you're advice to store your private key securely on github action secret

## Example Usage

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
          FLUTTER_VESION: "3.19.6"
          FLUTTER_BUILD_APK: false
          FLUTTER_BUILD_AAB: false
          
          SHOREBIRD_RELEASE: false
          SHOREBIRD_RELEASE_IOS: false
          SHOREBIRD_RELEASE_ANDROID: false

          SHOREBIRD_PATCH: true
          SHOREBIRD_PATCH_RELEASE_VERSION: "1.0.0+1"
          SHOREBIRD_PATCH_ANDROID: true
          SHOREBIRD_PATCH_IOS: false
          SHOREBIRD_TOKEN: ${{ secrets.SHOREBIRD_TOKEN }}

          ANDROID_SIGNING_PRIVATE_PEM: ${{ secrets.ANDROID_SIGNING_PRIVATE_PEM }}
