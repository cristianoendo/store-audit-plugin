# store-audit

Comprehensive App Store & Google Play Store compliance scanner for Claude Code.

Scans any app/SaaS codebase against live store guidelines to detect potential rejection causes before submission. Auto-detects your tech stack and applies automatic fixes for deterministic issues.

## Installation

Add to your Claude Code plugins:

```bash
claude plugins add /path/to/store-audit
```

Or clone from GitHub:

```bash
git clone https://github.com/cristianoendo/store-audit-plugin.git ~/.claude/plugins/store-audit
```

## Usage

```
/store-audit ios       # Audit for Apple App Store
/store-audit android   # Audit for Google Play Store
/store-audit all       # Audit for both (default)
```

## What It Checks

| Category | Examples |
|----------|---------|
| **Config & Permissions** | Info.plist, AndroidManifest, entitlements, permissions vs actual usage |
| **Privacy & Compliance** | Privacy policy, GDPR/LGPD, ATT, PrivacyInfo.xcprivacy, data safety, exposed API keys |
| **Payments & Subscriptions** | IAP requirement, restore purchases, subscription transparency, pricing, external links |
| **Content & UX** | Minimum functionality, Sign in with Apple, medical disclaimers, placeholder text, broken links |
| **Assets & Metadata** | Icons, splash screens, screenshots, store descriptions, keywords, age rating |
| **Build & Signing** | Code signing, target SDK, debug flags, ProGuard, deprecated APIs, binary size |

## How It Works

1. **Stack Detection** — auto-detects your framework (Capacitor, React Native, Flutter, Expo, native, PWA)
2. **Live Guidelines Fetch** — fetches current guidelines from Apple and Google official sites
3. **Parallel Audit** — 6 specialized auditors scan your codebase simultaneously
4. **Report & Fix** — generates severity-ranked report and applies automatic fixes

### Severity Levels

| Level | Meaning |
|-------|---------|
| CRITICO | Will cause rejection — explicit guideline violation |
| ALTO | High risk — interpretive guideline violation |
| MEDIO | Moderate risk — depends on reviewer |
| BAIXO | Best practice — improves approval chances |

## Supported Stacks

- Capacitor (iOS + Android)
- React Native / Expo
- Flutter
- Cordova
- Native iOS (Swift/Kotlin)
- Native Android
- PWA

## Output

Generates `store-audit-report-[platform]-YYYY-MM-DD.md` in your project root with:
- Severity-ranked findings with file paths and line numbers
- Specific guideline references
- Auto-fix summary (files modified)
- Passed checks list
- Next steps prioritization

## License

MIT
