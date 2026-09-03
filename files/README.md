# Placement Quest 3D — Android (TWA) build

This wraps your live web app (`https://quest-hire.preview.emergentagent.com`) into an
installable Android app using a **Trusted Web Activity**. It loads your real site inside
a full Chrome rendering engine, so the Three.js/WebGL scene runs exactly like it does
in a browser tab — no visible browser chrome, full-screen, launches from its own icon.

## Prerequisites (on your own machine)
- Node.js 16+
- Java JDK 17
- Android Studio (for the SDK + emulator/signing), or just the Android command-line SDK

## 1. Install Bubblewrap
```bash
npm install -g @bubblewrap/cli
```

## 2. Initialize the project
Use the `twa-manifest.json` included here directly:
```bash
mkdir quest-hire-android && cd quest-hire-android
# copy twa-manifest.json into this folder
bubblewrap init --manifest ./twa-manifest.json
```
Bubblewrap will download the Android SDK components it needs on first run and generate
a signing keystore (`android.keystore`) if one doesn't exist yet — **back this keystore
up somewhere safe**, you need the same one for every future update.

## 3. Verify domain ownership (required for full-screen mode)
Without this step, the app will fall back to showing a Chrome address bar.

1. Build once to generate the keystore, then get its fingerprint:
   ```bash
   keytool -list -v -keystore android.keystore -alias questhire
   ```
2. Copy the `SHA256` fingerprint into `assetlinks.json` (included here) in place of
   `PUT_YOUR_SHA256_FINGERPRINT_HERE`.
3. Host that file at exactly:
   `https://quest-hire.preview.emergentagent.com/.well-known/assetlinks.json`
   (this needs to be served by your backend/frontend host — ask whoever manages that
   deployment to add the static file if you can't add it yourself).

## 4. Build the app
```bash
bubblewrap build
```
This produces:
- `app-release-signed.apk` — install directly on a device for testing
- `app-release-bundle.aab` — the file you upload to Google Play

## 5. Test on a device
```bash
adb install app-release-signed.apk
```

## 6. Publish
Upload the `.aab` to the Google Play Console under a new app listing. You'll need:
- App icon (512×512) and feature graphic
- Screenshots (phone, and landscape since this manifest sets `orientation: landscape`)
- Privacy policy URL (required by Play even for apps with no accounts/data collection)

## Notes specific to this app
- **Orientation** is locked to `landscape` in the manifest since it's a 3D game — change
  to `"any"` if you want portrait support too, but you'll want to test the touch
  joystick/camera controls in portrait first.
- **Save data**: the app's `placement-quest-3d-save-v1` LocalStorage save will work fine
  inside the TWA since it's the same site/origin — progress carries over between
  browser and app if a user uses both.
- If WebGL performance feels different on low-end Android devices than in desktop
  Chrome, that's a device GPU limitation, not something the TWA wrapper introduces.

## Alternative if you don't want a Play Store listing
If you just want an installable file to share directly (sideload, not through Play),
you can skip Play Console and just distribute the signed `.apk` from step 4 — Android
will ask users to allow installs from unknown sources.
