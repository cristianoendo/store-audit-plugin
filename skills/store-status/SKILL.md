---
name: store-status
description: |
  Use this skill to check the current status of an app in the App Store and/or Google Play Store. Fetches review status, rejection reasons, and build information directly from the store APIs via Fastlane.
  Triggers on: "status da loja", "app status", "store status", "status App Store", "status Google Play", "como está o app na loja", "app review status", "check store", "verificar status", "qual o status", "app foi rejeitado?", "motivo da rejeição", "rejection reason".
allowed-tools: ["Read", "Bash", "Grep", "Glob"]
---

# Store Status Checker

Fetch the current review status and rejection details from App Store Connect and Google Play Console.

## Workflow

### Step 1: Detect Platforms and Credentials

Check which platforms are configured:

```bash
# iOS
test -d ios/ && echo "iOS: YES" || echo "iOS: NO"
test -f fastlane/keys/AuthKey_GL6GTMLNUW.p8 && echo "iOS API Key: YES" || echo "iOS API Key: NO"

# Android
test -d android/ && echo "Android: YES" || echo "Android: NO"
test -f fastlane/keys/google-play-key.json && echo "Android Key: YES" || echo "Android Key: NO"
```

If credentials are missing, report which ones and stop for that platform.

### Step 2: Fetch iOS App Status

Use Fastlane's Spaceship library to query App Store Connect API:

```bash
cd ios/App && bundle exec ruby -e '
require "spaceship"

# Connect using API key
key = Spaceship::ConnectAPI::Token.create(
  key_id: "GL6GTMLNUW",
  issuer_id: "144d062f-8485-4f60-b339-55daf6793405",
  filepath: "./fastlane/keys/AuthKey_GL6GTMLNUW.p8"
)
Spaceship::ConnectAPI.token = key

# Find the app
app = Spaceship::ConnectAPI::App.find("com.vidaleve.ciclo")

# Get latest version info
versions = app.get_app_store_versions
latest = versions.first

puts "=== iOS App Status ==="
puts "App: #{app.name}"
puts "Bundle ID: #{app.bundle_id}"
puts "Version: #{latest.version_string}"
puts "State: #{latest.app_store_state}"

# If rejected, get resolution center messages
if latest.app_store_state == "REJECTED" || latest.app_store_state == "DEVELOPER_REJECTED"
  # Get the app store version submission
  submissions = latest.get_app_store_version_submissions rescue []
  puts "Submission count: #{submissions.length}"

  # Try to get review details
  begin
    review = latest.get_app_store_review_detail
    if review
      puts "Contact Email: #{review.contact_email}"
      puts "Contact Phone: #{review.contact_phone}"
      puts "Notes: #{review.notes}"
    end
  rescue => e
    puts "Review detail error: #{e.message}"
  end
end

# Get builds
builds = latest.get_builds rescue []
if builds.any?
  build = builds.first
  puts "Latest Build: #{build.version}"
  puts "Build State: #{build.processing_state}"
end
'
```

**Alternative approach** if Spaceship fails — use Fastlane's `deliver` action:

```bash
cd ios/App && fastlane run deliver \
  api_key_path:"./fastlane/keys/api_key.json" \
  app_identifier:"com.vidaleve.ciclo" \
  skip_binary_upload:true \
  skip_screenshots:true \
  skip_metadata:true \
  precheck_include_in_app_purchases:false \
  force:true 2>&1 | head -50
```

**Fallback**: If API access fails entirely, instruct user to:
1. Open https://appstoreconnect.apple.com
2. Go to Apps → Vida Leve → App Review → Resolution Center
3. Copy the rejection message and paste it here

### Step 3: Fetch Android App Status

```bash
cd android && bundle exec ruby -e '
require "google/apis/androidpublisher_v3"

# Authenticate
service = Google::Apis::AndroidpublisherV3::AndroidPublisherService.new
key_file = "./fastlane/keys/google-play-key.json"

authorizer = Google::Auth::ServiceAccountCredentials.make_creds(
  json_key_io: File.open(key_file),
  scope: "https://www.googleapis.com/auth/androidpublisher"
)
service.authorization = authorizer

package = "com.vidaleve.ciclo"

puts "=== Android App Status ==="

# Get tracks info
["internal", "alpha", "beta", "production"].each do |track_name|
  begin
    track = service.get_edit_track(package, "auto", track_name)
    if track.releases&.any?
      release = track.releases.first
      puts "Track: #{track_name}"
      puts "  Status: #{release.status}"
      puts "  Version Codes: #{release.version_codes&.join(", ")}"
      puts "  Name: #{release.name}"
    end
  rescue => e
    # Track may not exist
  end
end
'
```

**Fallback**: If API access fails, instruct user to:
1. Open https://play.google.com/console
2. Go to App → Publishing overview
3. Check for any policy violations or review status

### Step 4: Read Current Build Info from Project Files

Always read local project files as supplementary info:

```bash
# iOS version/build
grep -A1 "CURRENT_PROJECT_VERSION" ios/App/App.xcodeproj/project.pbxproj | head -4
grep -A1 "MARKETING_VERSION" ios/App/App.xcodeproj/project.pbxproj | head -4

# Android version/build
grep "versionCode" android/app/build.gradle
grep "versionName" android/app/build.gradle
```

### Step 5: Generate Status Report

Present findings in this format:

```markdown
# Store Status Report — [Date]

## App: Vida Leve Ciclo (com.vidaleve.ciclo)

| Platform | Version | Build | Status | Notes |
|----------|---------|-------|--------|-------|
| iOS      | X.X.X   | N     | [State] | [Details] |
| Android  | X.X.X   | N     | [State] | [Details] |

## Rejection Details (if applicable)

### iOS — App Store Connect
- **Status**: REJECTED
- **Guidelines Cited**: [list]
- **Reviewer Notes**: [text]
- **Resolution Center Messages**: [text]

### Android — Google Play Console
- **Status**: [status]
- **Policy Violations**: [list]

## Recommended Next Steps
1. [Based on findings]
```

### Step 6: Suggest Next Actions

Based on the status:
- **REJECTED** or **UNRESOLVED_ISSUES**: **Stop.** Do NOT draft fixes inside the status report. Tell the user: "App is rejected — invoke `/store-audit:store-fix` BEFORE doing anything else. The Reply button in the Resolution Center disappears once the submission state changes, so the response must be drafted first." If the user explicitly opts to continue inline (rare), still load `store-fix/SKILL.md` first to enforce the response-first ordering.
- **WAITING_FOR_REVIEW**: "App is in queue. No action needed."
- **READY_FOR_SALE / PUBLISHED**: "App is live. Use `/store-audit:store-audit` for proactive compliance check."
- **DEVELOPER_REJECTED**: "You canceled the submission. Use `/store-audit:store-submit` when ready to resubmit."

## Spaceship API gaps (real-world fallbacks)

Some Connect endpoints have been deprecated or trail the UI. When the script fails or returns empty:

- **`v1/resolutionCenterThreads` returns 404** — Spaceship's `fetch_resolution_center_threads` no longer works. To read the actual rejection message text, use Playwright on `/apps/{app_id}/distribution/reviewsubmissions/details/{submission_id}` and extract the message body. The submission ID can be obtained from `app.get_in_progress_review_submission(platform: "IOS").id`.
- **`Build.all(version: "N")` returns empty for several minutes** even after upload completes — TestFlight UI surfaces the build first. To check if a freshly uploaded build is processed, prefer Playwright on `/teams/{team_id}/apps/{app_id}/testflight/ios` and look for the row marked "Concluído". Polling Spaceship `Build.all` every 60s wastes time when the UI already shows VALID.
- **Build attachment to a version is NOT automatic** — `upload_to_app_store` adds the binary to the app, but the version's "Compilação" relationship still points to the previous build. Either call `Spaceship::ConnectAPI::AppStoreVersion#select_build(build_id:)` explicitly, or delegate the swap to Playwright (see `../store-fix/references/resubmit-checklist.md`).

## Error Handling

- **Missing API key**: Report exactly which key is missing and where it should be placed
- **Expired token**: Suggest regenerating the API key in App Store Connect
- **Rate limit**: Wait and retry after 60 seconds
- **Network error**: Report and suggest checking internet connection
- **Ruby/Spaceship not available**: Fall back to Fastlane CLI commands or manual check instructions
