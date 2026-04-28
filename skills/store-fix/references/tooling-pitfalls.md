# Tooling Pitfalls (real failures from past sessions)

Concrete technical traps that have wasted real time. Read before reaching for Spaceship/Ruby/Fastlane.

## 1. Spaceship API trails the TestFlight UI

`Spaceship::ConnectAPI::Build.all(app_id:, version: "N", processing_states: ...)` returns an empty array for several minutes after `upload_to_app_store` reports success — even when the build is already visible as **"Concluído"** in the TestFlight web UI.

**Don't**: poll `Build.all` every 60s waiting for it to surface.

**Do**: open Playwright on `/teams/{team_id}/apps/{app_id}/testflight/ios` and grep the build row for "Concluído". The UI is the source of truth for processing status.

## 2. Resolution Center API is dead

`Spaceship::ConnectAPI::ReviewSubmission#fetch_resolution_center_threads` calls `GET /v1/resolutionCenterThreads`, which now returns 404 (Apple deprecated it). There is no public REST endpoint for the message thread.

**Do**: scrape `/apps/{app_id}/distribution/reviewsubmissions/details/{submission_id}` via Playwright. The message body lives inside `region "[author][date]"` accessible nodes.

## 3. Build attachment is NOT automatic

Uploading a binary via `upload_to_app_store` adds it to the app, but the in-flight version's `build` relationship still points to the old (rejected) build. Submitting "as is" sends the OLD binary to review again.

**Do**: explicitly call `AppStoreVersion#select_build(build_id:)` via Spaceship, OR delegate the swap to Playwright (delete row from "Compilação" table, click "Adicionar compilação", pick new build, "Concluído", "Salvar"). See `resubmit-checklist.md`.

**Verify**: the version page's "Compilação" table must show the new build number BEFORE clicking "Reenviar para Revisão".

## 4. `vite build` env-vars must be in `.env` at build time

`import.meta.env.VITE_FOO` is replaced by the LITERAL string at build time. If `VITE_FOO` is unset, Rollup emits `""` and the bundle ships with an empty key.

For RevenueCat/IAP: a missing `VITE_REVENUECAT_IOS_API_KEY` causes `Purchases.configure()` to silently no-op via `console.warn`, and `getOfferings()` then throws — which the App Review team experiences as "an error message was displayed and we were unable to purchase any subscription" (Guideline 2.1(b)).

**Do**: before `npx cap sync`, run:

```bash
grep -oE "appl_[a-zA-Z0-9]+|goog_[a-zA-Z0-9]+" dist/assets/*.js | head -3
```

Empty result = the key wasn't bundled. Check `.env` in the repo root (gitignored), not just worktrees or Vercel's env panel.

## 5. Stdout buffering kills `run_in_background` Ruby scripts

A backgrounded `ruby script.rb 2>&1 | head -30` will appear empty for the entire run. `head` does line-buffer its output, but Ruby's `puts` block-buffers stdout when stdout is a pipe — output sits in the buffer until the pipe closes (process death).

**Don't**: pipe long-running Ruby scripts through `head`, `cat`, or any aggregator while running in background.

**Do**: either
- add `STDOUT.sync = true` as the first line of the script, or
- run the script directly without a tail aggregator: `ruby script.rb 2>&1 > log.txt`, and `Monitor` the log file with `grep --line-buffered`.

## 6. `fastlane lanes` resolves the Fastfile from the CWD

Running `fastlane lanes` from `ios/App/` finds a different (sometimes empty) Fastfile via fastlane's directory walking, returning lanes that don't exist when run from project root.

**Do**: always run fastlane from the directory that contains `fastlane/Fastfile` (typically the repo root for Capacitor projects).

## 7. Apple metadata length limits silently fail in deliver

`fastlane deliver` does NOT trim metadata — it errors out late in the upload. Limits as of 2026:

| Field | Max chars |
|-------|-----------|
| `name.txt` | 30 |
| `subtitle.txt` | 30 |
| `keywords.txt` | 100 (commas count) |
| `promotional_text.txt` | 170 |
| `description.txt` | 4000 |
| `release_notes.txt` | 4000 |

**Do**: before `fastlane metadata`, run a length check:

```bash
for f in fastlane/metadata/pt-BR/{name,subtitle,keywords,promotional_text}.txt; do
  echo "$f: $(wc -c < "$f") chars"
done
```

## 8. Promotional Image vs Review Screenshot are different fields

In the IAP product page (`/apps/{id}/distribution/subscriptions/{sub_id}`), there are **two** image upload areas:

- **"Imagem (opcional)"** — the promotional image used for App Store promotion / win-back / offer code redeem. Apple's Guideline 2.3.2 applies here. Must NOT be a screenshot from the app.
- **"Captura de tela"** under "Informações para a equipe de revisão" — internal reference for reviewers. Screenshots are explicitly expected here.

**Do**: when removing a 2.3.2-flagged image, scroll past the first image upload to make sure you're not deleting the reviewer screenshot by mistake.
