# store-audit

Comprehensive App Store & Google Play Store compliance toolkit for Claude Code.

Covers the complete submission lifecycle: pre-submission audit → status check → fix rejections → submit. Auto-detects your stack, fetches live guidelines from official sites, and integrates with Fastlane for end-to-end automation.

## Features

### Pre-Submission Audit
- 6 specialized auditors (config, privacy, payments, content/UX, assets, build) running in parallel
- Live guidelines fetch from Apple and Google official sites
- Auto-detection of project stack (Capacitor, React Native, Flutter, Expo, Cordova, native, PWA)
- Severity-ranked report with auto-fixes for deterministic issues

### Rejection Lifecycle
- Status check via App Store Connect and Google Play Console APIs
- Response drafting for the review team (before canceling submission)
- Code/config fixes with build verification
- Fastlane integration for build, upload, and submission

## Installation

Already installed locally at `~/.claude/plugins/local/store-audit/`. Or clone from GitHub:

```bash
git clone https://github.com/cristianoendo/store-audit-plugin.git ~/.claude/plugins/local/store-audit
```

## Commands & Skills

| Command / Skill | Purpose |
|-----------------|---------|
| `/store-audit [ios\|android\|all]` | Pre-submission audit with 6 parallel auditors |
| `store-status` skill | Check current store status and rejection reasons via APIs |
| `store-fix` skill | Analyze rejection, draft response, apply fixes, verify |
| `store-submit` skill | Pre-flight + build + upload + submit via Fastlane |

## Pipeline

```
Pre-submission                Pre-submission       Submit              If rejected
     │                              │                  │                   │
     ▼                              ▼                  ▼                   ▼
/store-audit              ──── (review)  ────  store-submit  ─────  store-status
(parallel auditors                                                        │
 + live guidelines                                                        ▼
 + auto-fixes)                                                       store-fix
                                                                     (response +
                                                                      code fix)
                                                                          │
                                                                          ▼
                                                                     store-submit
                                                                     (resubmit)
```

## What the Audit Checks

| Category | Examples |
|----------|----------|
| **Config & Permissions** | Info.plist, AndroidManifest, entitlements, permissions vs actual usage |
| **Privacy & Compliance** | Privacy policy, GDPR/LGPD/COPPA, ATT, PrivacyInfo.xcprivacy, data safety, exposed API keys |
| **Payments & Subscriptions** | IAP requirement, restore purchases, transparency, pricing, external links |
| **Content & UX** | Minimum functionality, Sign in with Apple, medical disclaimers, placeholder text, broken links |
| **Assets & Metadata** | Icons, splash screens, screenshots, store descriptions, keywords, age rating |
| **Build & Signing** | Code signing, target SDK, debug flags, ProGuard, deprecated APIs, binary size |

## Severity Levels

| Level | Meaning |
|-------|---------|
| CRITICO | Will cause rejection — explicit guideline violation |
| ALTO | High risk — interpretive guideline violation |
| MEDIO | Moderate risk — depends on reviewer |
| BAIXO | Best practice — improves approval chances |

## Requirements

### For Audit (`/store-audit`)
- Node.js / project dependencies installed
- Internet (for live guidelines fetch — falls back to knowledge cutoff)

### For Status / Submit / Fix Skills
- **Fastlane** (`gem install fastlane` or via Bundler)
- **App Store Connect API Key** (`.p8` file) in `fastlane/keys/`
- **Google Play Service Account** (JSON key) in `fastlane/keys/`
- **Xcode** (for iOS builds)
- **Android SDK** (for Android builds)

## Supported Stacks

Capacitor (iOS + Android), React Native / Expo, Flutter, Cordova, Native iOS (Swift), Native Android (Kotlin), PWA

## Output

Audit generates `store-audit-report-[platform]-YYYY-MM-DD.md` in the project root with:
- Severity-ranked findings with file paths and line numbers
- Specific guideline references
- Auto-fix summary (files modified)
- Passed checks list
- Next steps prioritization

## License

MIT
