---
description: Scan your app/SaaS codebase for App Store and Google Play Store rejection causes. Fetches live guidelines and generates a severity-ranked report with auto-fixes.
argument-hint: "[ios|android|all]"
allowed-tools:
  - Task
  - TodoWrite
  - Read
  - Bash
---

# /store-audit

You are the orchestrator for a comprehensive App Store and Google Play Store compliance audit.

## Parse Arguments

```
$ARGUMENTS → platform
```

- `ios` → audit for Apple App Store only
- `android` → audit for Google Play Store only
- `all` or empty → audit for both platforms (default)

Set `platform` based on the argument. If no argument provided, default to `all`.

## Execution Flow

### Phase 1: Reconnaissance (parallel)

Create a TodoWrite with the full audit pipeline, then launch TWO agents in parallel in a SINGLE message:

**Agent 1 — Stack Detector**:
```
Detect the technology stack of the project in the current working directory. 
Report as structured JSON including: nativeFramework, webFramework, platforms (ios/android/pwa), paymentSystems, backend, sensitiveData categories, languages, cicd tools, appId, appName, version info.
Examine: package.json, pubspec.yaml, build.gradle, Podfile, capacitor.config.*, app.json, Info.plist, AndroidManifest.xml, ios/ and android/ directories.
```
Use subagent: `store-audit:stack-detector`

**Agent 2 — Guidelines Fetcher**:
```
Fetch the latest store review guidelines for platform: [PLATFORM].
Organize by category: Safety, Performance, Business/Payments, Design, Legal/Privacy, Metadata, Kids.
Include specific guideline IDs and what to check in code.
If fetch fails, use knowledge cutoff and mark with STALE warning.
```
Use subagent: `store-audit:guidelines-fetcher`

Wait for both to complete. Extract the stack profile JSON and the organized guidelines.

### Phase 2: Audit (parallel)

Launch ALL 6 auditor agents in a SINGLE message (parallel execution). Each receives the stack profile + their relevant guideline section + the platform:

**Agent 3 — Config & Permissions Auditor** (`store-audit:config-permissions-auditor`):
```
Audit configuration files and permissions for platform: [PLATFORM].
Stack profile: [STACK_JSON]
Relevant guidelines: [CONFIG/PERFORMANCE GUIDELINES]
```

**Agent 4 — Privacy & Compliance Auditor** (`store-audit:privacy-compliance-auditor`):
```
Audit privacy policies, data collection, compliance for platform: [PLATFORM].
Stack profile: [STACK_JSON]
Relevant guidelines: [PRIVACY/LEGAL GUIDELINES]
```

**Agent 5 — Payments & Subscriptions Auditor** (`store-audit:payments-auditor`):
```
Audit payment implementation and subscription flows for platform: [PLATFORM].
Stack profile: [STACK_JSON]
Relevant guidelines: [BUSINESS/PAYMENTS GUIDELINES]
```

**Agent 6 — Content & UX Auditor** (`store-audit:content-ux-auditor`):
```
Audit content, UX flows, and app completeness for platform: [PLATFORM].
Stack profile: [STACK_JSON]
Relevant guidelines: [DESIGN/CONTENT GUIDELINES]
```

**Agent 7 — Assets & Metadata Auditor** (`store-audit:assets-metadata-auditor`):
```
Audit app assets and store metadata for platform: [PLATFORM].
Stack profile: [STACK_JSON]
Relevant guidelines: [METADATA/LISTING GUIDELINES]
```

**Agent 8 — Build & Signing Auditor** (`store-audit:build-signing-auditor`):
```
Audit build configuration and signing for platform: [PLATFORM].
Stack profile: [STACK_JSON]
Relevant guidelines: [TECHNICAL REQUIREMENTS]
```

Wait for all 6 to complete. Collect all findings.

### Phase 3: Consolidation

Launch the Report Consolidator agent:

**Agent 9 — Report Consolidator** (`store-audit:report-consolidator`):
```
Consolidate these audit findings into a final report for platform: [PLATFORM].
Stack profile: [STACK_JSON]
Guidelines source: [live/cutoff]

Findings from all auditors:
[ALL_FINDINGS]

1. Deduplicate findings
2. Classify severity (CRITICO/ALTO/MEDIO/BAIXO)
3. Apply auto-fixes for deterministic issues
4. Generate report at: store-audit-report-[platform]-[YYYY-MM-DD].md
```

### Phase 4: Summary

After the consolidator finishes, display a summary to the user:

```
## Store Audit Complete

**Platform**: [iOS/Android/Both]
**Report**: store-audit-report-[platform]-YYYY-MM-DD.md

### Summary
| Severity | Count |
|----------|-------|
| CRITICO  | N     |
| ALTO     | N     |
| MEDIO    | N     |
| BAIXO    | N     |

### Auto-fixes Applied: N
[List of files modified]

### Top Priority Items
1. [Most critical finding]
2. [Second most critical]
3. [Third most critical]
```

## Progress Tracking

Update TodoWrite throughout execution:
1. "Detect project stack" → in_progress → completed
2. "Fetch store guidelines" → in_progress → completed
3. "Audit config & permissions" → in_progress → completed
4. "Audit privacy & compliance" → in_progress → completed
5. "Audit payments & subscriptions" → in_progress → completed
6. "Audit content & UX" → in_progress → completed
7. "Audit assets & metadata" → in_progress → completed
8. "Audit build & signing" → in_progress → completed
9. "Consolidate report & apply fixes" → in_progress → completed
