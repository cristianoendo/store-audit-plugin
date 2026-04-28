# Android Pre-Submission Checklist

## Build Configuration

- [ ] `targetSdkVersion` meets Google Play requirement (API 34+)
- [ ] `minSdkVersion` set appropriately (21+ for most apps)
- [ ] `compileSdkVersion` matches or exceeds targetSdkVersion
- [ ] App built as AAB (not APK): `./gradlew bundleRelease`
- [ ] ProGuard/R8 configured for release: `minifyEnabled true`
- [ ] Signing keystore secure and not committed to git
- [ ] Version code incremented from previous submission

## AndroidManifest.xml

- [ ] No unused permissions declared
- [ ] `INTERNET` permission present (for networked apps)
- [ ] No `ACCESS_FINE_LOCATION` without location feature
- [ ] No `CAMERA` without camera feature
- [ ] `android:usesCleartextTraffic="false"` (HTTPS only)
- [ ] Health Connect permissions match actual usage

## Google Play Console

- [ ] Data Safety section complete and accurate
- [ ] Privacy policy URL provided
- [ ] App category correct
- [ ] Content rating questionnaire completed
- [ ] Target audience selected correctly
- [ ] App access instructions provided for review (if login required)

## In-App Purchases

- [ ] Google Play Billing Library or RevenueCat configured
- [ ] Products created in Google Play Console with correct pricing
- [ ] Test purchases verified in sandbox
- [ ] External payment methods gated on Android native
- [ ] Subscription management/cancellation accessible

## Content & Design

- [ ] No placeholder content ("TODO", "Lorem ipsum", test data)
- [ ] Screenshots match current app UI
- [ ] App description accurate and up-to-date
- [ ] Tablet layout works (if app runs on tablets)
- [ ] Medical disclaimers present (if health app)

## Privacy

- [ ] Account deletion available from settings
- [ ] Privacy policy accessible in-app
- [ ] Data collection matches Data Safety declaration
- [ ] No API keys hardcoded in client code

## Build & Upload Commands

```bash
npm run build                    # Web assets compile
npx cap sync android             # Capacitor sync
cd android && ./gradlew bundleRelease  # Build AAB

# Upload via Fastlane
fastlane android upload_metadata       # Metadata only
fastlane android upload_screenshots    # Screenshots only
```

## Fastlane Lanes Reference

| Lane | Command | Purpose |
|------|---------|---------|
| `upload_screenshots` | `fastlane android upload_screenshots` | Upload screenshots + images |
| `upload_metadata` | `fastlane android upload_metadata` | Upload text metadata |

## Google Play Console Checklist

- [ ] AAB uploaded to correct track (internal/beta/production)
- [ ] Release notes provided (localized)
- [ ] Data Safety form up to date
- [ ] Content rating current
- [ ] Store listing (title, description, screenshots) up to date
- [ ] Privacy policy URL valid and accessible
- [ ] Target countries/regions selected
- [ ] Managed publishing enabled (if gradual rollout)
