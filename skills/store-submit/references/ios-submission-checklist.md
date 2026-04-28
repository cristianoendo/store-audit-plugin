# iOS Pre-Submission Checklist — Detailed

## Info.plist Checks

### Permissions (NSUsageDescription keys)
Only include permissions for features with **fully implemented native plugins**:

| Key | When Required |
|-----|---------------|
| `NSCameraUsageDescription` | Camera capture feature implemented |
| `NSPhotoLibraryUsageDescription` | Photo picker feature implemented |
| `NSCalendarsUsageDescription` | Calendar integration via native plugin |
| `NSHealthShareUsageDescription` | HealthKit read via native plugin |
| `NSHealthUpdateUsageDescription` | HealthKit write via native plugin |
| `NSLocationWhenInUseUsageDescription` | Location feature implemented |
| `NSUserTrackingUsageDescription` | ATT implemented AND tracking declared |

**Check**: `grep -c "NSUsageDescription" ios/App/App/Info.plist` — verify each has native implementation.

### Device Capabilities
```xml
<key>UIRequiredDeviceCapabilities</key>
<array>
    <string>arm64</string>  <!-- CORRECT -->
</array>
```
**NEVER**: `armv7` (32-bit, deprecated 2017)

### Encryption
```xml
<key>ITSAppUsesNonExemptEncryption</key>
<false/>  <!-- For apps using only standard HTTPS -->
```

## PrivacyInfo.xcprivacy Checks

### Required API Types

| API Category | Reason Code | When Required |
|--------------|-------------|---------------|
| `NSPrivacyAccessedAPICategoryUserDefaults` | `CA92.1` | Almost always (Capacitor uses it) |
| `NSPrivacyAccessedAPICategoryFileTimestamp` | `C617.1` | If checking file dates |
| `NSPrivacyAccessedAPICategoryDiskSpace` | `E174.1` | If checking disk space |

### Collected Data Types

| SDK | Data Type to Declare |
|-----|---------------------|
| RevenueCat | `NSPrivacyCollectedDataTypePurchaseHistory` |
| Supabase Auth | `NSPrivacyCollectedDataTypeEmailAddress` |
| Health features | `NSPrivacyCollectedDataTypeHealthData` |
| Analytics | `NSPrivacyCollectedDataTypePerformanceData` |

### Tracking
- `NSPrivacyTracking` must be `false` if not tracking
- If `false`, NO data type should have "Used for tracking" in App Store Connect

## Subscription Configuration

### Subscription Group
1. Subscriptions → Group → **Language section**
2. Must have at least one localization with Subscription Group Display Name
3. WITHOUT this, subscriptions show "Missing Metadata"

### Each Subscription Product
Required fields:
- Reference name
- Product ID (matches RevenueCat/StoreKit)
- Duration (1 month, 1 year, etc.)
- Price configured for all territories
- At least one localization (name + description)
- Review screenshot (1290x2796, 1242x2208, or 1170x2532)
- Review notes

### Linking to App Version
- App version page → "In-App Purchases and Subscriptions" section
- Click "Select In-App Purchases or Subscriptions"
- Check both subscription products → Done
- REQUIRED for first submission of subscriptions

## Xcode Project
- `CURRENT_PROJECT_VERSION` incremented from previous submission
- `MARKETING_VERSION` matches App Store version
- Code signing configured for distribution
- Swift Package dependencies resolved

## Build & Upload Commands
```bash
npm run build              # Web assets compile
npx cap sync ios           # Capacitor sync
cd ios/App && fastlane release       # Build + upload
cd ios/App && fastlane submit_for_review  # Submit for review
```

## Fastlane Lanes Reference

| Lane | Command | Purpose |
|------|---------|---------|
| `screenshots` | `cd ios/App && fastlane screenshots` | Upload screenshots |
| `metadata` | `cd ios/App && fastlane metadata` | Upload metadata |
| `release` | `cd ios/App && fastlane release` | Build IPA + upload |
| `submit_for_review` | `cd ios/App && fastlane submit_for_review` | Submit for review |
| `upload_all` | `cd ios/App && fastlane upload_all` | Screenshots + metadata |

## App Store Connect Checklist

- [ ] Version string set correctly
- [ ] Build uploaded and processed
- [ ] Screenshots for all required device sizes
- [ ] App description (localized)
- [ ] Keywords (localized)
- [ ] Privacy policy URL
- [ ] Support URL
- [ ] App Review Information (login credentials, notes)
- [ ] Age rating questionnaire completed
- [ ] Privacy labels match PrivacyInfo.xcprivacy
- [ ] IAP products linked to version
- [ ] Export compliance declarations set
