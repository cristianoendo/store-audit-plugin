---
name: store-submit
description: |
  Use this skill to build, upload, and submit the app to the App Store and/or Google Play Store via Fastlane. Handles pre-flight checks, build number increments, and the full submission workflow.
  Triggers on: "submeter app", "submit app", "enviar para loja", "upload to store", "release", "deploy app", "publicar app", "submeter para review", "submit for review", "fastlane release", "upload build", "resubmeter", "resubmit".
allowed-tools: ["Read", "Bash", "Grep", "Glob", "Task", "TodoWrite"]
---

# Store Submission

Build, upload, and submit the app to the App Store and/or Google Play via Fastlane.

## Syntax

`/store-audit:store-submit [ios|android|all]`

- `ios` — Submit to Apple App Store only
- `android` — Submit to Google Play Store only
- `all` (default) — Submit to both stores

## Workflow

### Step 1: Pre-Flight Checks

Run ALL checks before proceeding. If any fails, STOP and report.

1. **Quick audit** — Run `/store-audit [ios|android]` for a complete pre-submission audit. Must have zero CRITICO findings.

2. **Build check**:
   ```bash
   npm run build
   ```
   Must complete with zero errors.

3. **Test check**:
   ```bash
   npx vitest run
   ```
   All tests must pass.

4. **Git status**:
   ```bash
   git status
   ```
   Working tree must be clean (no uncommitted changes). If dirty, ask user to commit or stash first.

5. **API keys**:
   ```bash
   # iOS
   test -f fastlane/keys/AuthKey_GL6GTMLNUW.p8 && echo "iOS key: OK" || echo "iOS key: MISSING"

   # Android
   test -f fastlane/keys/google-play-key.json && echo "Android key: OK" || echo "Android key: MISSING"
   ```

6. **Fastlane available**:
   ```bash
   which fastlane || bundle exec fastlane --version
   ```

Report pre-flight status:
```markdown
## Pre-Flight Check
| Check | Status |
|-------|--------|
| Audit (zero CRITICAL) | ✅/❌ |
| Build (npm run build) | ✅/❌ |
| Tests (vitest) | ✅/❌ |
| Git clean | ✅/❌ |
| API keys | ✅/❌ |
| Fastlane | ✅/❌ |
```

### Step 2: Increment Build Number

**iOS**:
```bash
# Read current build number
grep "CURRENT_PROJECT_VERSION" ios/App/App.xcodeproj/project.pbxproj | head -1

# Increment (N → N+1) in ALL occurrences
```

**Android**:
```bash
# Read current version code
grep "versionCode" android/app/build.gradle

# Increment (N → N+1)
```

Dispatch `store-audit:store-submitter` agent to handle the increments.

### Step 3: Build & Upload

**iOS**:
```bash
# Sync web assets to native project
npm run build
npx cap sync ios

# Build IPA and upload to App Store Connect
cd ios/App && fastlane release
```

**Android**:
```bash
# Sync web assets to native project
npm run build
npx cap sync android

# Build AAB
cd android && ./gradlew bundleRelease
```

### Step 4: Submit for Review (iOS)

After the build is processed in App Store Connect:
```bash
cd ios/App && fastlane submit_for_review
```

### Step 5: Post-Submission Checklist

Present this checklist to the user:

```markdown
## Post-Submission Checklist

### App Store Connect (iOS)
- [ ] Verify build appears and is processed
- [ ] Link IAP subscriptions to the version (if applicable)
- [ ] Verify subscription group has localization
- [ ] Verify review screenshots have correct dimensions
- [ ] Verify privacy labels match PrivacyInfo.xcprivacy
- [ ] Check App Review Information has login credentials

### Google Play Console (Android)
- [ ] Verify AAB is uploaded to correct track
- [ ] Verify Data Safety section is up to date
- [ ] Verify privacy policy link is present
- [ ] Verify content rating is current

### Git
- [ ] Commit build number increment
- [ ] Push changes
- [ ] Create PR if on feature branch
```

### Step 6: Commit & Push

```bash
git add ios/App/App.xcodeproj/project.pbxproj android/app/build.gradle
git commit -m "chore: bump build numbers for store submission"
```

### Step 7: Report Final Status

```markdown
## Submission Report — [Date]

| Platform | Version | Build | Status |
|----------|---------|-------|--------|
| iOS      | X.X.X   | N     | Uploaded / Submitted for Review |
| Android  | X.X.X   | N     | Uploaded / Published |

### Next Steps
- iOS: Aguardando revisão (~24-48h)
- Android: [Status do track]
```

## Metadata-Only Updates

For uploading just metadata or screenshots (no new binary):

### iOS
```bash
cd ios/App && fastlane metadata          # Metadata only
cd ios/App && fastlane screenshots       # Screenshots only
cd ios/App && fastlane upload_all        # Both
```

### Android
```bash
fastlane android upload_metadata         # Metadata
fastlane android upload_screenshots      # Screenshots
```

## Error Recovery

| Error | Cause | Fix |
|-------|-------|-----|
| "No binary found" | Build didn't produce IPA/AAB | Check build output, signing config |
| "Invalid binary" | Binary processing failed | Check Xcode version, signing certs |
| "Unauthorized" | API key invalid/expired | Regenerate key in ASC/Google Console |
| "Version already exists" | Same version+build submitted | Increment build number |
| "App in wrong state" | Can't submit in current state | Check current version state first |

## References

- **`references/ios-submission-checklist.md`** — Detailed iOS checklist
- **`references/android-submission-checklist.md`** — Detailed Android checklist
