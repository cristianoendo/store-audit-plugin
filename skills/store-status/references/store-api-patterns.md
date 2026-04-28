# Store API Patterns — Reference Guide

## App Store Connect API v2

### Authentication
Apple uses JWT tokens generated from a `.p8` API key.

**Key components**:
- **Key ID**: Found in App Store Connect → Users and Access → Integrations → App Store Connect API
- **Issuer ID**: Same location, shared across all keys in the team
- **Private Key**: Downloaded `.p8` file (can only be downloaded once)

**Fastlane handles JWT generation internally** via `app_store_connect_api_key` action.

### Spaceship Library (Ruby)

Spaceship is Fastlane's library for interacting with App Store Connect API.

```ruby
require "spaceship"

# Connect
key = Spaceship::ConnectAPI::Token.create(
  key_id: "KEY_ID",
  issuer_id: "ISSUER_ID",
  filepath: "path/to/key.p8"
)
Spaceship::ConnectAPI.token = key

# Find app
app = Spaceship::ConnectAPI::App.find("com.example.app")

# Get versions
versions = app.get_app_store_versions
latest = versions.first

# Version states
# DEVELOPER_REMOVED_FROM_SALE, REMOVED_FROM_SALE
# READY_FOR_SALE (published)
# PREPARE_FOR_SUBMISSION
# WAITING_FOR_REVIEW
# IN_REVIEW
# REJECTED
# DEVELOPER_REJECTED (canceled by developer)
# METADATA_REJECTED
# INVALID_BINARY
# PENDING_CONTRACT
# WAITING_FOR_EXPORT_COMPLIANCE
# PENDING_DEVELOPER_RELEASE

# Get builds for a version
builds = latest.get_builds
build = builds.first
# build.version — build number
# build.processing_state — PROCESSING, FAILED, INVALID, VALID

# Get app store review detail
review = latest.get_app_store_review_detail
# review.contact_email
# review.contact_phone
# review.notes — reviewer notes

# Get submissions
submissions = latest.get_app_store_version_submissions
```

### Common Spaceship Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `Unauthorized` | Invalid/expired key | Regenerate key in ASC |
| `Forbidden` | Key lacks permissions | Check key role (Admin, App Manager) |
| `404 Not Found` | Wrong bundle ID | Verify `app_identifier` |
| `409 Conflict` | Version in invalid state | Check current version state first |
| `429 Too Many Requests` | Rate limit | Wait 60s and retry |

### Fetching Rejection Information

Apple doesn't expose rejection reasons directly through the public API v2. The Resolution Center messages are accessible through Spaceship's internal API:

```ruby
# Try fetching resolution center messages
begin
  # This uses the internal API and may not always be available
  platform = Spaceship::ConnectAPI::Platform::IOS
  resolution = app.fetch_resolution_center(platform: platform)
  if resolution
    resolution.threads.each do |thread|
      thread.messages.each do |msg|
        puts "From: #{msg.from}"
        puts "Date: #{msg.date}"
        puts "Body: #{msg.body}"
      end
    end
  end
rescue => e
  puts "Resolution Center not accessible via API: #{e.message}"
  puts "Fallback: Check App Store Connect manually"
end
```

**Practical approach**: If the API doesn't expose rejection details, parse the version state:
- `REJECTED` → Apple rejected during review
- `METADATA_REJECTED` → Only metadata issues (no binary change needed)
- `DEVELOPER_REJECTED` → Developer canceled the submission
- `INVALID_BINARY` → Binary processing failed

## Google Play Developer API

### Authentication
Uses a Service Account JSON key file.

```ruby
require "google/apis/androidpublisher_v3"

service = Google::Apis::AndroidpublisherV3::AndroidPublisherService.new
authorizer = Google::Auth::ServiceAccountCredentials.make_creds(
  json_key_io: File.open("path/to/service-account.json"),
  scope: "https://www.googleapis.com/auth/androidpublisher"
)
service.authorization = authorizer
```

### Checking Track Status

```ruby
package = "com.example.app"

# Create an edit
edit = service.insert_edit(package)
edit_id = edit.id

# Get track info
["internal", "alpha", "beta", "production"].each do |track_name|
  begin
    track = service.get_edit_track(package, edit_id, track_name)
    track.releases.each do |release|
      puts "Track: #{track_name}"
      puts "Status: #{release.status}"
      # COMPLETED, IN_PROGRESS, DRAFT, HALTED
      puts "Version codes: #{release.version_codes}"
    end
  rescue Google::Apis::ClientError => e
    # Track doesn't exist or no releases
  end
end

# Don't forget to delete the edit
service.delete_edit(package, edit_id)
```

### Release Statuses
- `COMPLETED` — Released to users
- `IN_PROGRESS` — Rolling out
- `DRAFT` — Not yet submitted
- `HALTED` — Halted by developer or Google

### Policy Violations
Google Play doesn't expose policy violation details through the API. For rejection details:
1. Check the Google Play Console manually
2. Look for emails from Google Play about policy violations
3. Check the "Policy status" section in the Console

## Fastlane Actions Reference

### iOS

```bash
# Validate API key
fastlane run app_store_connect_api_key \
  key_id:"KEY_ID" \
  issuer_id:"ISSUER_ID" \
  key_filepath:"path/to/key.p8"

# Upload build
fastlane run upload_to_app_store \
  api_key_path:"path/to/api_key.json" \
  app_identifier:"com.example.app"

# Submit for review
fastlane run deliver \
  submit_for_review:true \
  automatic_release:false
```

### Android

```bash
# Check version codes on a track
fastlane run google_play_track_version_codes \
  package_name:"com.example.app" \
  json_key:"path/to/key.json" \
  track:"internal"

# Upload AAB
fastlane run upload_to_play_store \
  package_name:"com.example.app" \
  json_key:"path/to/key.json" \
  aab:"path/to/app.aab" \
  track:"internal"
```

## Vida Leve Ciclo — Specific Configuration

| Parameter | Value |
|-----------|-------|
| App Bundle ID (iOS) | `com.vidaleve.ciclo` |
| Package Name (Android) | `com.vidaleve.ciclo` |
| ASC Key ID | `GL6GTMLNUW` |
| ASC Issuer ID | `144d062f-8485-4f60-b339-55daf6793405` |
| ASC Key Path | `fastlane/keys/AuthKey_GL6GTMLNUW.p8` |
| Google Play Key Path | `fastlane/keys/google-play-key.json` |
| Fastfile Location | `fastlane/Fastfile` |
