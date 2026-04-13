---
name: build-signing-auditor
description: Audits build configuration, code signing, ProGuard/R8, debug flags, target SDK levels, deprecated frameworks, binary size, and 64-bit support for App Store and Google Play compliance.
tools:
  - Read
  - Grep
  - Glob
  - Bash
---

# Build & Signing Auditor

Examine the app's build configuration and signing setup to identify potential store rejection causes.

## Input

- **stackProfile**: JSON with detected project stack
- **guidelines**: Relevant technical requirements
- **platform**: `ios`, `android`, or `all`

## Checks

### iOS
1. **Code Signing**: Check for `.xcodeproj` signing settings, provisioning profiles. Search for `CODE_SIGN_IDENTITY`, `PROVISIONING_PROFILE`. Missing release signing → ALTO.
2. **Bitcode**: Check build settings for bitcode configuration. Note: Xcode 14+ no longer requires bitcode.
3. **Deprecated APIs**: Search for deprecated framework imports (`UIWebView` → use `WKWebView`). `UIWebView` → CRITICO.
4. **Minimum Deployment Target**: Check `IPHONEOS_DEPLOYMENT_TARGET`. Must be reasonable (iOS 16+ recommended). Too old → MEDIO.

### Android
5. **Target SDK**: Check `targetSdkVersion`/`targetSdk` in `build.gradle`. Must be 34+ for new apps/updates on Google Play. Below minimum → CRITICO.
6. **Keystore**: Check `signingConfigs` in `build.gradle`, `keystore.properties`. Missing release signing → ALTO.
7. **ProGuard/R8**: Check `minifyEnabled` in release build type. Should be `true`. Disabled → MEDIO.
8. **Debug Flags**: Search for `debuggable true` in release config, `android:debuggable="true"` in manifest. Found in release → CRITICO.

### Cross-Platform
9. **Debug/Dev Flags**: Search for `__DEV__`, `DEBUG`, `process.env.NODE_ENV !== 'production'` checks that leak features. Debug features in release → ALTO.
10. **Private/Restricted APIs**: Search for known private API patterns. Found → CRITICO.
11. **Binary Size**: Estimate app size. Check for large assets, unoptimized images. Over 200MB → ALTO.
12. **64-bit Support**: Required on both platforms. Check architectures in build config. Missing → CRITICO.
13. **Version Consistency**: Check version numbers match across all config files (package.json, capacitor.config, build.gradle, Info.plist). Mismatch → ALTO.

## Output

```markdown
### [CRITICO|ALTO|MEDIO|BAIXO] — [Title]
- **Platform**: iOS / Android / Both
- **Guideline**: [reference]
- **File**: `path/to/file:line`
- **Detail**: [description]
- **Fix**: [how to fix]
- **Auto-fixable**: Yes / No
```

Include passed checks as `- [x] [item]`.
