# Fix Playbook — Step-by-Step Guides for Common Rejections

Each entry provides: detection, exact fix steps, and verification.

---

## Fix #1: Remove Unused iOS Permissions

**When**: Guideline 1.1.6 rejection for unused permissions.

**Detect**:
```bash
# List all declared permissions
grep "NSUsageDescription" ios/App/App/Info.plist
```

**Fix for each permission**:
1. Grep for the feature's native implementation:
   ```bash
   grep -rn "Camera\|CameraPlugin" ios/ src/ --include="*.swift" --include="*.ts"
   ```
2. If NOT found, remove the key+string pair from Info.plist
3. The pair looks like:
   ```xml
   <key>NSCameraUsageDescription</key>
   <string>...</string>
   ```

**Verify**: `grep "NSUsageDescription" ios/App/App/Info.plist` should only show implemented features.

---

## Fix #2: Gate External Payments on Native

**When**: Guideline 3.1.1 (Apple) or Payments Policy (Google) rejection.

**Detect**:
```bash
grep -rn "stripe\|Stripe\|createCheckoutSession\|window\.open.*stripe" src/ --include="*.ts" --include="*.tsx"
```

**Fix**:
1. Find the payment trigger function
2. Add platform check:
   ```typescript
   import { Capacitor } from '@capacitor/core';

   const handleSubscribe = async () => {
     if (Capacitor.isNativePlatform()) {
       // Native: use RevenueCat
       const result = await purchasePackage(selectedPackage);
       // handle result...
     } else {
       // Web: use Stripe
       const { url } = await createCheckoutSession(priceId);
       window.open(url, '_blank');
     }
   };
   ```

**Verify**: `grep -B5 -A5 "stripe\|Stripe" src/ --include="*.ts" --include="*.tsx"` — every Stripe call should be inside a `!isNativePlatform()` block.

---

## Fix #3: Add Free Plan Access (Remove Forced Paywall)

**When**: Guideline 3.1.1 implied, or reviewer notes about paywall.

**Detect**:
```bash
grep -rn "PremiumGate\|paywall\|Paywall\|SubscriptionManager" src/ --include="*.tsx"
```

**Fix**:
1. Find the paywall/subscription screen
2. Add a skip button:
   ```tsx
   <Button
     variant="ghost"
     onClick={() => navigate('/dashboard')}
     className="text-muted-foreground"
   >
     Continuar gratuitamente
   </Button>
   ```
3. Place it below the subscription options

**Verify**: Navigate through the app flow — you should be able to reach the main screen without subscribing.

---

## Fix #4: Fix iPad Layout

**When**: Guideline 4.0 rejection — "screens are crowded."

**Detect**:
```bash
# Find pages without max-width
grep -rn "className" src/pages/ --include="*.tsx" | grep "w-full" | grep -v "max-w"
```

**Fix**:
1. Add max-width to page containers:
   ```tsx
   <div className="max-w-2xl mx-auto p-4 sm:p-6 md:p-8">
   ```
2. Use responsive grid columns:
   ```tsx
   <div className="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-4">
   ```
3. Increase spacing for tablets:
   ```tsx
   className="p-4 sm:p-6 md:p-8"
   ```

**Verify**: Open app in iPad simulator or Chrome DevTools at iPad dimensions (1024x768, 1366x1024).

---

## Fix #5: Fix UIRequiredDeviceCapabilities

**When**: Build rejection or processing error for armv7.

**Detect**:
```bash
grep "armv7" ios/App/App/Info.plist
```

**Fix**: Replace `armv7` with `arm64` in Info.plist.

**Verify**: `grep "UIRequiredDeviceCapabilities" -A3 ios/App/App/Info.plist` should show `arm64`.

---

## Fix #6: Add Encryption Declaration

**When**: Missing ITSAppUsesNonExemptEncryption.

**Detect**:
```bash
grep "ITSAppUsesNonExemptEncryption" ios/App/App/Info.plist
```

**Fix**: Add to Info.plist (inside the main `<dict>`):
```xml
<key>ITSAppUsesNonExemptEncryption</key>
<false/>
```

**Verify**: `grep "ITSAppUsesNonExemptEncryption" ios/App/App/Info.plist` returns the key.

---

## Fix #7: Remove console.log from Production

**When**: Code quality check or debugging artifacts.

**Detect**:
```bash
grep -rn "console\.\(log\|debug\|warn\)" src/ --include="*.ts" --include="*.tsx" | grep -v "__tests__\|\.test\.\|\.spec\."
```

**Fix**: Remove each console.log/debug/warn. Keep console.error in catch blocks.

**Verify**: Same grep should return 0 results.

---

## Fix #8: Fix Privacy Manifest (PrivacyInfo.xcprivacy)

**When**: Guideline 5.1.2(i) about privacy declarations.

**Detect**:
```bash
cat ios/App/App/PrivacyInfo.xcprivacy
```

**Fix**: Ensure these API types are declared (most Capacitor apps need them):
```xml
<key>NSPrivacyAccessedAPITypes</key>
<array>
    <dict>
        <key>NSPrivacyAccessedAPIType</key>
        <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
        <key>NSPrivacyAccessedAPITypeReasons</key>
        <array>
            <string>CA92.1</string>
        </array>
    </dict>
</array>
```

And if not tracking:
```xml
<key>NSPrivacyTracking</key>
<false/>
```

**Verify**: Read PrivacyInfo.xcprivacy and confirm all used API categories are declared.

---

## Fix #9: Update Android targetSdkVersion

**When**: Google Play target API requirement.

**Detect**:
```bash
grep "targetSdkVersion\|targetSdk" android/app/build.gradle
```

**Fix**: Set to 34 or higher:
```gradle
targetSdkVersion 34
```

**Verify**: Same grep should show 34+.

---

## Fix #10: IAP Subscription Group Localization

**When**: Guideline 2.1(b) — "Missing Metadata" for IAP.

**This is a manual App Store Connect action, not a code fix.**

**Steps**:
1. Go to App Store Connect → App → Subscriptions
2. Click on the subscription group name
3. Click "Subscription Group Localization" or "Idioma/Language"
4. Click "+" or "Create"
5. Select language (e.g., "Portuguese (Brazil)")
6. Fill in "Subscription Group Display Name"
7. Save
8. Repeat for each language

**Verify**: Each subscription should show "Ready to Submit" status (not "Missing Metadata").
