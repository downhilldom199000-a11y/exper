# TKREC APK Inventory

**Generated:** 2026-08-26  
**Analyst:** Automated inventory via JADX 1.5.6 + apktool 3.0.2  
**Original file:** `~/Desktop/tkk/TKREC 1.3.1.apk+`  
**Note:** This is an **Android App Bundle (AAB)** packaged as `.apk+`, not a standard APK.

---

## 1. Basic Information

| Field | Value |
|---|---|
| **Package name** | `com.tkrec.app` |
| **Application name** | TKREC |
| **Version name** | 1.3.1 |
| **Version code** | 39 |
| **Min SDK** | 24 (Android 7.0 Nougat) |
| **Target/Compile SDK** | 36 (Android 16) |
| **Bundle format** | AAB (`apk+` type) |
| **Source stamp** | `STAMP_TYPE_DISTRIBUTION_APK` |
| **Source store** | https://play.google.com/store |

---

## 2. Bundle Components

The file `TKREC 1.3.1.apk+` is an Android App Bundle containing:

| Component | Size | Description |
|---|---|---|
| `base.apk` | 52.2 MB | Main app module (code, resources, assets) |
| `split_config.arm64_v8a.apk` | 788 KB | Native libraries for ARM64 |
| `split_config.en.apk` | 68 KB | English locale resources |
| `split_config.xxhdpi.apk` | 160 KB | Extra-extra-high-density resources |
| `icon.png` | 20 KB | App icon |
| `apk+.json` | 113 B | Bundle metadata |

---

## 3. Signing Certificate

| Field | Value |
|---|---|
| **Stamp cert SHA256** | `3257d599a49d2c961a471ca9843f59d341a405884583fc087df4237b733bbd6d` |
| **Base APK hash (SHA256)** | `a9bd6df3e626a20f8f50a3967804dd5ca9c72991086b57ff17eb523d407639db` |
| **Jar signing** | Not a signed JAR (split APK — signed at bundle level) |

---

## 4. DEX Files

| File | Size |
|---|---|
| `classes.dex` | 8.6 MB |
| `classes2.dex` | 9.4 MB |
| `classes3.dex` | 8.0 MB |
| `classes4.dex` | 6.7 MB |
| `classes5.dex` | 10.0 MB |
| **Total** | **5 DEX files, ~42.7 MB** |

**Smali file counts:** 42,168 files across 5 smali directories.  
**JADX Java file count:** 21,508 `.java` files.

---

## 5. Native Libraries (.so)

All native libraries are in the `arm64-v8a` architecture split:

| Library | Size | Purpose |
|---|---|---|
| `libsentry.so` | 706.8 KB | Sentry native crash reporting |
| `libsentry-android.so` | 16.4 KB | Sentry Android integration |
| `libdatastore_shared_counter.so` | 6.9 KB | AndroidX DataStore |

**Note:** No other architectures present (armeabi-v7a, x86, x86_64 absent).

---

## 6. Permissions (17 total)

| Permission | Category |
|---|---|
| `INTERNET` | Network |
| `ACCESS_NETWORK_STATE` | Network |
| `POST_NOTIFICATIONS` | Notifications |
| `WAKE_LOCK` | System |
| `VIBRATE` | System |
| `FOREGROUND_SERVICE` | System |
| `FOREGROUND_SERVICE_DATA_SYNC` | System |
| `READ_EXTERNAL_STORAGE` | Storage |
| `WRITE_EXTERNAL_STORAGE` | Storage |
| `USE_CREDENTIALS` | Auth |
| `USE_BIOMETRIC` | Auth |
| `USE_FINGERPRINT` | Auth |
| `com.google.android.gms.permission.AD_ID` | Advertising |
| `ACCESS_ADSERVICES_AD_ID` | Advertising |
| `ACCESS_ADSERVICES_ATTRIBUTION` | Advertising |
| `ACCESS_ADSERVICES_TOPICS` | Advertising |
| `ACCESS_ADSERVICES_CUSTOM_AUDIENCE` | Advertising |
| `com.google.android.c2dm.permission.RECEIVE` | Messaging |
| `com.android.vending.BILLING` | Billing |
| `com.google.android.finsky.permission.BIND_GET_INSTALL_REFERRER_SERVICE` | Install referrer |

**Custom permission declared:**  
- `com.tkrec.app.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION` (signature level)

---

## 7. Components

### 7.1 Activities (10)

| Activity | Exported | Notes |
|---|---|---|
| `com.tkrec.app.MainActivity` | **Yes** | Main launcher, `singleTask`, portrait |
| `com.tkrec.app.ShareReceiverActivity` | **Yes** | Handles `ACTION_SEND` (text/plain) |
| `com.tkrec.video.player.VideoPlayerActivity` | No | PiP support |
| `ee.forgr.capacitor.social.login.TwitterLoginActivity` | No | Twitter OAuth WebView |
| `ee.forgr.capacitor.social.login.OAuth2LoginActivity` | No | Generic OAuth2 WebView |
| `com.google.android.gms.auth.api.signin.internal.SignInHubActivity` | No | Google Sign-In |
| `com.google.android.gms.ads.AdActivity` | No | AdMob |
| `com.google.android.gms.ads.OutOfContextTestingActivity` | No | AdMob testing |
| `com.google.android.gms.ads.NotificationHandlerActivity` | No | AdMob notifications |
| `com.google.android.gms.common.api.GoogleApiActivity` | No | Google API |
| `com.revenuecat.purchases.amazon.purchasing.ProxyAmazonBillingActivity` | No | Amazon IAP |
| `com.revenuecat.purchases.SimulatedStoreErrorDialogActivity` | No | RevenueCat |
| `com.android.billingclient.api.ProxyBillingActivity` | No | Play Billing |
| `com.android.billingclient.api.ProxyBillingActivityV2` | No | Play Billing |
| `com.google.android.play.core.common.PlayCoreDialogWrapperActivity` | No | Play Core |
| `androidx.credentials.playservices.HiddenActivity` | No | Credentials |
| `androidx.credentials.playservices.IdentityCredentialApiHiddenActivity` | No | Credentials |

### 7.2 Services (11)

| Service | Exported | Notes |
|---|---|---|
| `com.capacitorjs.plugins.pushnotifications.MessagingService` | No | FCM push |
| `com.grec.capacitor.downloader.service.DownloadForegroundService` | No | Foreground download, `dataSync` type |
| `com.google.firebase.messaging.FirebaseMessagingService` | No | FCM core |
| `com.google.firebase.components.ComponentDiscoveryService` | No | Firebase DI |
| `com.google.android.gms.measurement.AppMeasurementService` | No | Analytics |
| `com.google.android.gms.measurement.AppMeasurementJobService` | No | Analytics |
| `com.google.android.gms.ads.AdService` | No | AdMob |
| `com.android.vending.billing.InAppBillingService.BIND` | (query) | Play Billing |
| `androidx.sharetarget.ChooserTargetServiceCompat` | **Yes** | Share target |
| `com.google.android.gms.auth.api.signin.RevocationBoundService` | **Yes** | Google auth |
| `androidx.work.impl.background.systemjob.SystemJobService` | **Yes** | WorkManager |
| `androidx.work.impl.foreground.SystemForegroundService` | No | WorkManager |
| `androidx.room.MultiInstanceInvalidationService` | No | Room DB |
| `com.google.android.datatransport.runtime.backends.TransportBackendDiscovery` | No | Transport |
| `com.google.android.datatransport.runtime.scheduling.jobscheduling.JobInfoSchedulerService` | No | Transport |

### 7.3 Receivers (13)

| Receiver | Exported | Notes |
|---|---|---|
| `com.grec.capacitor.downloader.service.DownloadActionReceiver` | No | Download cancel actions |
| `com.google.firebase.iid.FirebaseInstanceIdReceiver` | **Yes** | FCM token refresh |
| `com.google.android.gms.measurement.AppMeasurementReceiver` | No | Analytics |
| `com.amazon.device.iap.ResponseReceiver` | **Yes** | Amazon IAP |
| `com.facebook.CurrentAccessTokenExpirationBroadcastReceiver` | No | Facebook SDK |
| `com.facebook.AuthenticationTokenManager$CurrentAuthenticationTokenChangedBroadcastReceiver` | No | Facebook SDK |
| `androidx.work.impl.utils.ForceStopRunnable$BroadcastReceiver` | No | WorkManager |
| `androidx.work.impl.diagnostics.DiagnosticsReceiver` | **Yes** | WorkManager diagnostics |
| `androidx.profileinstaller.ProfileInstallReceiver` | **Yes** | Baseline profiles |
| Various WorkManager ConstraintProxy receivers | No | System state monitors |
| `com.google.android.datatransport.runtime.scheduling.jobscheduling.AlarmManagerSchedulerBroadcastReceiver` | No | Transport |

### 7.4 Providers (6)

| Provider | Exported | Authority |
|---|---|---|
| `androidx.core.content.FileProvider` | No | `com.tkrec.app.fileprovider` |
| `com.google.android.gms.ads.MobileAdsInitProvider` | No | `com.tkrec.app.mobileadsinitprovider` |
| `com.google.firebase.provider.FirebaseInitProvider` | No | `com.tkrec.app.firebaseinitprovider` |
| `androidx.startup.InitializationProvider` | No | `com.tkrec.app.androidx-startup` |
| `io.sentry.android.core.SentryInitProvider` | No | `com.tkrec.app.SentryInitProvider` |
| `io.sentry.android.core.SentryPerformanceProvider` | No | `com.tkrec.app.SentryPerformanceProvider` |

---

## 8. Intent Filters & Deep Links

### Deep Links
- **Custom scheme:** `tkrec://` → `MainActivity`
- **HTTPS App Links (auto-verified):** `https://app.tkrec.com` → `MainActivity`

### Share Targets
- `ACTION_SEND` (text/plain) → `ShareReceiverActivity`

### Queried Intents
- `android.intent.action.VIEW` (https scheme)
- `CustomTabsService`
- `ACTION_INSERT` (calendar event)
- `ACTION_VIEW` (sms scheme)
- `ACTION_DIAL` (tel:)
- `InAppBillingService.BIND`
- `BillingOverrideService.BIND`

---

## 9. Network Security

| Setting | Value |
|---|---|
| **Cleartext traffic** | **ENABLED** (`android:usesCleartextTraffic="true"`) |
| **network_security_config.xml** | Not present (no custom config) |
| **Cordova config** | `<access origin="*" />` — allows all origins |

**Security note:** Cleartext HTTP is allowed. No certificate pinning config detected at the manifest/XML level.

---

## 10. Web Technologies

This is a **Capacitor hybrid app** — the core UI is web-based.

| Asset | Size | Purpose |
|---|---|---|
| `index.html` | 1.6 KB | Web entry point |
| `index-1.3.0.js` | 1.2 MB | Bundled application JS |
| `vendor-1.3.0.js` | 1.4 MB | Vendor/library bundle |
| `polyfills-1.3.0.js` | 15 KB | Browser polyfills |
| `index-1.3.0.css` | 786 KB | Application CSS |
| `native-bridge.js` | 51.9 KB | Capacitor native bridge |
| `manifest.webmanifest` | 970 B | PWA manifest |

**Framework7** icon fonts are bundled (Framework7Icons-Regular).

---

## 11. Capacitor Plugins (24 registered)

| Plugin | Package |
|---|---|
| AdMob | `@capacitor-community/admob` |
| In-App Review | `@capacitor-community/in-app-review` |
| Firebase Analytics | `@capacitor-firebase/analytics` |
| App | `@capacitor/app` |
| App Launcher | `@capacitor/app-launcher` |
| Clipboard | `@capacitor/clipboard` |
| Device | `@capacitor/device` |
| Haptics | `@capacitor/haptics` |
| Preferences | `@capacitor/preferences` |
| Push Notifications | `@capacitor/push-notifications` |
| Splash Screen | `@capacitor/splash-screen` |
| Status Bar | `@capacitor/status-bar` |
| Navigation Bar | `@capgo/capacitor-navigation-bar` |
| Social Login | `@capgo/capacitor-social-login` |
| OTA Plugin | `@ota/plugin` |
| RevenueCat Purchases | `@revenuecat/purchases-capacitor` |
| Sentry | `@sentry/capacitor` |
| Device Info | `capacitor-deviceinfo` |
| Native Share | `capacitor-native-share` |
| Video Player | `capacitor-video-player` |
| CPD Downloader | `cpd` |
| TKREC Attribution | `tkrec-attribution-sdk` |
| Facebook App Events | `tkrec-facebook-app-events-sdk` |
| TikTok Business SDK | `tkrec-tiktok-business-sdk` |

---

## 12. Third-Party SDKs Detected

| SDK | Purpose |
|---|---|
| Google Play Services / GMS | Core Android services |
| Firebase (Messaging, Analytics, Installations) | Push notifications, analytics |
| Google AdMob | In-app advertising |
| Google Play Billing (v8.0.0) | In-app purchases |
| RevenueCat | Subscription management |
| Facebook SDK | Analytics, events |
| TikTok Business SDK | Attribution, events |
| Sentry | Crash reporting, performance |
| OkHttp3 / Okio | HTTP networking |
| Amazon IAP | Amazon app store purchases |
| Capacitor | Hybrid app runtime |
| Framework7 | UI framework (icon fonts) |

---

## 13. Security-Relevant Findings (Preliminary)

| # | Finding | Severity |
|---|---|---|
| 1 | `usesCleartextTraffic="true"` — HTTP allowed | Medium |
| 2 | `Cordova config.xml: access origin="*"` — WebView allows all origins | Medium |
| 3 | OAuth2 WebView activities with JavaScript enabled | Info |
| 4 | Social login (Google, Apple) via WebView | Info |
| 5 | No network_security_config.xml present | Medium |
| 6 | `allowBackup="true"` — app data extractable via ADB | Low |
| 7 | OTA plugin present (`@ota/plugin`) — potential update mechanism | Info |
| 8 | Multiple advertising/attribution SDKs (AdMob, Facebook, TikTok) | Info |

---

## 14. Decompiled File Locations

| Resource | Path |
|---|---|
| **JADX output (Java)** | `analysis/jadx_base/sources/` |
| **apktool output (smali + resources)** | `analysis/apktool_base/` |
| **Manifest** | `analysis/apktool_base/AndroidManifest.xml` |
| **Native libs (arm64)** | `analysis/split_arm64/lib/arm64-v8a/` |
| **Evidence copy** | `analysis/evidence/tkrec.apk` |
| **File tree** | `analysis/static/FILE_TREE.txt` |
| **This inventory** | `analysis/APK_INVENTORY.md` |

---

## 15. Summary

| Metric | Value |
|---|---|
| APK path | `~/Desktop/tkk/TKREC 1.3.1.apk+` |
| Package name | `com.tkrec.app` |
| Version | 1.3.1 (code 39) |
| DEX files | 5 |
| Native libraries | 3 (arm64-v8a only) |
| JADX succeeded | Yes (21,508 Java files) |
| apktool succeeded | Yes (with resource reference warnings) |
| Decompilation type | AAB bundle → base.apk decompiled |
| App type | Capacitor hybrid (web + native bridge) |
