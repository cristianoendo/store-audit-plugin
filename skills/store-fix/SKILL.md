---
name: store-fix
description: |
  Use this skill when an app has been rejected by the App Store or Google Play and needs to be fixed. Analyzes the rejection, drafts a response for the review team, applies code fixes, and verifies the fixes.
  Triggers on: "corrigir rejeição", "fix rejection", "resolver rejeição", "fix app store", "fix google play", "aplicar correções", "apply fixes", "app rejeitado", "app rejected", "guideline violation", "violação de guideline", "rejeição", "rejection".
allowed-tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob", "Task", "TodoWrite"]
---

# Store Rejection Fixer

Analyzes app store rejections, drafts a response to the review team, applies code/config fixes, and verifies everything before resubmission.

## CRITICAL RULE

**Draft and send the response to the review team BEFORE canceling the submission or making code changes.** The "Reply" button in the Resolution Center disappears after canceling a submission. Always:

1. Draft response → 2. User sends via Resolution Center → 3. Then fix code → 4. Then resubmit

## Workflow

### Step 1: Gather Rejection Context

Accept rejection details from one of these sources:

1. **From `/store-audit:store-status`**: Structured rejection data from API
2. **Pasted rejection message**: User pastes the text from Resolution Center
3. **Guideline numbers**: User provides specific guidelines (e.g., "2.1(b), 4.0")
4. **Free-form description**: User describes the issue ("rejected for IAP issues")

Parse and extract:
- Platform (iOS / Android)
- Guideline numbers cited
- Issue descriptions
- Reviewer notes
- Review device details (if mentioned)

### Step 2: Map Rejection to Code

For each guideline violation:

1. **Read the reference**: Load `../store-audit/references/apple-review-guidelines.md` or `../store-audit/references/google-play-policies.md`
2. **Find the pattern**: Load `../store-audit/references/rejection-fix-patterns.md` and find the matching pattern
3. **Run the detection commands**: Execute the pattern's detection script to find affected files
4. **Classify the fix**:
   - **Auto-fix**: Deterministic, single correct resolution (e.g., remove unused permission)
   - **Guided fix**: Requires judgment but has clear patterns (e.g., add platform guard)
   - **Manual fix**: Requires App Store Connect / Google Play Console action (e.g., IAP localization)

Present a fix plan:

```markdown
## Fix Plan

### Auto-Fixes (will apply immediately)
| # | Issue | File | Fix |
|---|-------|------|-----|

### Guided Fixes (will apply with documentation)
| # | Issue | File | Fix |
|---|-------|------|-----|

### Manual Actions (you need to do these in the store console)
| # | Issue | Where | Action |
|---|-------|-------|--------|
```

### Step 3: Draft Response — BEFORE Any Code Changes

Use `references/response-templates.md` to draft a formal response:

1. Select the appropriate template (Apple English, Apple Portuguese, Google)
2. Fill in each guideline section with:
   - The exact guideline number and category
   - Root cause explanation (specific, honest)
   - Description of the fix being applied
3. Add proactive improvements section (if any)
4. Add closing with build number reference

Present the response to the user:

```
📋 Response draft for Resolution Center:

---
[Response text]
---

Please send this response via the Resolution Center BEFORE I start making code changes.
The Reply button disappears after canceling the submission.

Confirm when sent, and I'll proceed with the fixes.
```

**WAIT for user confirmation before proceeding.**

### Step 4: Create Fix Branch

```bash
git checkout -b fix/store-rejection-$(date +%Y-%m-%d)
```

### Step 5: Apply Fixes

Dispatch `store-audit:store-fixer` agent with the structured fix plan.

The agent will:
1. Apply auto-fixes first (low risk)
2. Apply guided fixes with documentation
3. Run `npm run build` after each group
4. Commit with conventional format

### Step 6: Verify Fixes

After all fixes are applied:

1. **Full build check**:
   ```bash
   npm run build
   ```

2. **Test suite**:
   ```bash
   npx vitest run
   ```

3. **Targeted re-audit**: Run `/store-audit [ios|android]` to re-audit the codebase, OR dispatch the specific auditor agent (`store-audit:config-permissions-auditor`, `store-audit:privacy-compliance-auditor`, `store-audit:payments-auditor`, `store-audit:content-ux-auditor`, `store-audit:assets-metadata-auditor`, `store-audit:build-signing-auditor`) most relevant to the violated guidelines. Verify zero findings for those guidelines.

4. **Report results**:

```markdown
## Fix Verification Report

### Fixes Applied
| # | Issue | Fix | Status |
|---|-------|-----|--------|
| 1 | [issue] | [fix] | ✅ Applied |

### Build: ✅ PASS
### Tests: ✅ PASS (X/X)
### Re-Audit: ✅ 0 CRITICAL findings

### Manual Actions Remaining
| # | Action | Where |
|---|--------|-------|
```

### Step 7: Suggest Next Steps

- If all fixes verified: "Correções aplicadas e verificadas. Use `/store-audit:store-submit` para submeter."
- If manual actions remain: "Correções de código aplicadas. Complete as ações manuais listadas acima antes de submeter."
- If re-audit finds new issues: "Novas issues encontradas durante a re-auditoria. Corrigindo..."

## Key Lessons (from Real Rejections)

1. **IAP "Missing Metadata"** — 90% of the time it's the subscription group missing localization, NOT missing screenshots
2. **Reply BEFORE resubmitting** — Reply button disappears after canceling
3. **iPad layout** — Apple reviewers always test iPad too
4. **Unused permissions** — Immediate rejection (Guideline 1.1.6)
5. **Privacy labels mismatch** — NSPrivacyTracking must match App Store Connect
6. **Privacy labels** — Do NOT mark tracking if you're not tracking

## References

- **`references/fix-playbook.md`** — Step-by-step playbook for common rejection fixes
- **`references/response-templates.md`** — Response templates for review teams
