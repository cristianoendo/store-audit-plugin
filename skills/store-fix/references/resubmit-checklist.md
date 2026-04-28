# iOS Resubmit Runbook (Playwright-first)

Step-by-step procedure for resubmitting a rejected iOS app **after** the response has been sent and code/metadata fixes are committed. Every step has been verified end-to-end against App Store Connect's current UI (Brazilian Portuguese — adapt labels for other locales).

> **Prerequisite**: Steps 1–3 from the `store-fix` Pre-Flight checklist are done (response sent, branch created, fixes applied & verified). The Playwright session is authenticated to `appstoreconnect.apple.com`.

## 0. Confirm reply was sent

Navigate to `/apps/{app_id}/distribution/reviewsubmissions/details/{submission_id}`.

Verify:
- The "Mensagens (N)" header shows `N >= 2` (Apple's last message + your reply).
- The most recent message author is the developer, not Apple.

If the most recent message is still from Apple, **stop** and go back to `store-fix` Step 2 (Draft & Send Response). Do not proceed.

## 1. Confirm new build is VALID via TestFlight UI

Do **not** rely on `Spaceship::ConnectAPI::Build.all(version: "N")` — it can return empty for several minutes after the build is already processed. Instead:

```text
Navigate to: /teams/{team_id}/apps/{app_id}/testflight/ios
```

Find the row matching your new build number. The status column should read **"Concluído"**. If it shows "Processando" or no row exists yet, wait 60–120s and refresh.

Once Concluído, capture the build's UUID from the row's link (`/testflight/ios/{build_uuid}/metadata`) — it's needed when the build picker dialog identifies builds by UUID rather than version.

## 2. Swap the attached build on the in-flight version

```text
Navigate to: /apps/{app_id}/distribution/ios/version/inflight
```

Find the **"Compilação"** section heading. The table beneath shows the currently attached build (likely the rejected one).

### 2a. Delete the old build

The "Apagar" button next to the old build can be off-screen or have a stale ARIA ref. If `browser_click` on the ref fails with `element is not visible`, run this fallback via `browser_evaluate`:

```js
() => {
  const tables = Array.from(document.querySelectorAll('table'));
  for (const t of tables) {
    if (t.innerText.includes('Compilação')) {
      const btn = t.querySelector('button[aria-label="Apagar"]');
      if (btn) { btn.click(); return { clicked: true }; }
    }
  }
  return { clicked: false };
}
```

After the click, the row should disappear and an "Adicionar compilação" button appears in its place. No confirmation dialog is shown for this delete.

### 2b. Add the new build

Click "Adicionar compilação". A modal "Adicione uma compilação" opens listing every processed build. Click the radio for the new build number, then click **"Concluído"**.

Verify the table now shows the new build number (e.g. "12") and version (e.g. "1.0").

### 2c. Save the version

The page-level **"Salvar"** button (top of the version page) should be enabled. Click it. After save:
- "Salvar" becomes disabled (no pending changes).
- **"Atualizar revisão"** becomes enabled.

## 3. Update the review submission

Click **"Atualizar revisão"**. This redirects you to `/apps/{app_id}/distribution/reviewsubmissions/details/{submission_id}`. The item row should now show `1.0 (NEW_BUILD)` with status **"Pronto para revisão"** and the **"Reenviar para Revisão do app"** button should be enabled.

## 4. Resubmit

Click **"Reenviar para Revisão do app"**. App Store Connect does not show a confirmation dialog for resubmit (different from cancel/delete). After ~3–5s the page refreshes.

Verify:
- The submission status banner reads **"Aguardando revisão"** (not "Problemas não resolvidos").
- The item status column reads **"Aguardando revisão"**.

## 5. Final sanity

```bash
# (via Spaceship, optional)
ruby -e '...; sub = app.get_in_progress_review_submission(platform: "IOS"); p sub.state'
# Expected: "WAITING_FOR_REVIEW"
```

If it still says `UNRESOLVED_ISSUES`, the resubmit click did not complete — re-run from Step 3.

## Common gotchas

- **"Salvar" stays disabled after attaching the new build** → you forgot to delete the old build first; the version still has both attached or the change wasn't registered. Refresh and try again.
- **"Atualizar revisão" stays disabled after Salvar** → there are other unsaved blockers on the version page (often a missing/invalid screenshot or removed encryption flag). Scroll the page for red error icons.
- **"Reenviar para Revisão do app" stays disabled on the submission page** → the version isn't actually saved. Go back to the version page and click Salvar again.
