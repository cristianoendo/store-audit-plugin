---
name: store-audit
description: "Use when the user mentions app store submission, app review, app rejection, store compliance, Play Store policy, App Store guidelines, store rejection, review rejection, app submission, submit to store, publish app, or wants to verify their app before submitting to Apple App Store or Google Play Store."
allowed-tools:
  - SlashCommand
  - Read
---

# Store Audit — App Store & Google Play Compliance Scanner

This skill detects when a user is preparing to submit an app to the App Store or Google Play, or has received a rejection, and suggests running a comprehensive compliance audit.

## When This Activates

This skill triggers when the user mentions:
- Submitting an app to the App Store or Google Play
- App review or app rejection
- Store compliance or guidelines
- Preparing for store submission
- App review feedback or rejection reasons

## What To Do

1. **Acknowledge** the user's intent (submission preparation or rejection troubleshooting)
2. **Suggest** running the `/store-audit` command:

> I can run a comprehensive compliance scan of your codebase against the latest App Store and Google Play guidelines. This will check configuration, privacy, payments, content/UX, assets, metadata, and build settings — and auto-fix deterministic issues.
>
> Run: `/store-audit ios`, `/store-audit android`, or `/store-audit all`

3. **If the user agrees**, invoke the `/store-audit` command via the Skill tool with the appropriate platform argument.

## What The Audit Covers

- **Config & Permissions**: Info.plist, AndroidManifest, entitlements, permissions vs usage
- **Privacy & Compliance**: Privacy policy, GDPR/LGPD, ATT, data safety, exposed API keys
- **Payments**: IAP requirement, restore purchases, subscription transparency, pricing
- **Content & UX**: Minimum functionality, Sign in with Apple, disclaimers, placeholder text, broken links
- **Assets & Metadata**: Icons, screenshots, store descriptions, age rating, localization
- **Build & Signing**: Code signing, target SDK, debug flags, deprecated APIs, binary size

Guidelines are fetched live from Apple and Google's official sites for maximum accuracy.
