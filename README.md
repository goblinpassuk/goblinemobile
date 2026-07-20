# GoblinPass Companion

GoblinPass Companion is a completely offline Android app for storing identifier-to-service mappings such as `001 → Amazon`. It is deliberately unable to model passwords, PINs, recovery keys, or other secrets.

## Security design

- The app requests no internet permission and contains no analytics, advertising, or tracking SDKs.
- Mapping data is held in `EncryptedSharedPreferences` using AES-256-SIV for keys and AES-256-GCM for values. The master key is created by Jetpack Security and protected by Android Keystore.
- The UI starts locked on every Activity creation and launcher re-entry. Android `BiometricPrompt` requires an enrolled fingerprint or face; device-credential fallback is deliberately disabled. The app locks after 30 seconds in the background.
- `FLAG_SECURE` blocks ordinary screenshots, screen recording, and readable recent-app previews on supported Android versions.
- Android cloud backup and device-transfer extraction are both explicitly disabled.
- Portable backups use PBKDF2-HMAC-SHA256 (310,000 iterations, random 128-bit salt) and AES-256-GCM (random 96-bit nonce). GCM authenticates the contents and format version. Restore validates the complete file before replacing local data.
- Decrypted entries are removed from Compose observable state when the lock screen returns. Password character arrays and plaintext backup byte arrays are overwritten after use where the platform permits.

`FLAG_SECURE` and a biometric UI gate reduce casual disclosure but cannot protect a compromised/rooted device or malicious accessibility service. Users should install only on a supported, patched Android device with a secure lock screen.

## Build

Requirements:

- Android Studio Ladybug (2024.2.1) or newer
- JDK 17 (Android Studio's bundled JDK is suitable)
- Android SDK 35

Steps:

1. Open this directory in Android Studio.
2. Allow Gradle to sync and install SDK 35 if prompted.
3. Select an Android device or emulator running Android 6.0 (API 23) or newer. Enrol a fingerprint or supported face unlock.
4. Run the `app` configuration.

From a terminal with `JAVA_HOME` and the Android SDK configured:

```text
./gradlew assembleDebug
```

For a production APK or App Bundle, create and protect your own signing key, configure release signing outside source control, then run `assembleRelease` or `bundleRelease`. Release builds enable code and resource shrinking.

## Use

- Tap **Add mapping** and enter an ID, service name, and optional category.
- Search matches IDs, service names, and categories.
- Tap an entry to edit it. Swipe it to request deletion, then confirm.
- Use the top-right menu to create or restore an encrypted `.gpc` backup. Backup passwords must be at least 12 characters and cannot be recovered.

The app begins with an empty list. Example mappings are `001 → Amazon`, `002 → Facebook`, `003 → Gmail`, and `004 → Barclays`; these examples are not preloaded so no data is assumed on the user's behalf.

## Project structure

- `data/` — validated mapping model, strict JSON codec, and encrypted repository
- `security/` — authenticated password-encrypted backup format
- `ui/` — Material 3 Compose UI, search, CRUD, category, and backup flows
- `MainActivity.kt` — screenshot protection, BiometricPrompt, and lifecycle auto-lock

No permissions beyond `USE_BIOMETRIC` are declared.
