---
name: signup-backend-dependencies
description: EMAP (epd-emap) backend endpoint powering Flow Option 3, and the existing route reused for Flow Option 2
---

# Backend Endpoints — Flow Options 2 & 3

Status as of **2026-09-01**, verified against `origin/staging` of `epd-emap`.

---

## 1. Flow Option 2 (Step 1 only, redirect to EMAP) — no new backend code

EMAP already has an admin-facing "Resume Merchant Signup" button (on the deal edit page, e.g. `/dashboard/applications/{uuid}/edit`) that just links to:

```
GET {EMAP_APP_URL}/upload-document/{uuid}?redirect=1
```

This route (`SignupController::uploadDocument`, `routes/web.php`) requires no auth — it logs the applicant in by `uuid` alone and, with `redirect=1`, forwards to `/signup/signature/{uuid}`, which routes the merchant to wherever their application currently stands (back into the form if incomplete, to signing if a template applies, or to the dashboard if already signed).

The partner-hosted Step 1 form already has everything it needs for this: `POST /v1/signup` returns the `uuid`. Just redirect the browser there after Step 1 succeeds — no `/v1/signup/continue` endpoint, no new token type, no backend changes at all.

**Known limitation (not something this skill's flow controls)**: `Application.step_count` is only ever recorded starting from Step 2's submission (`SignupAPIController::submitApplicationStep`) — the Step 1→2 transition in EMAP's own SPA happens client-side without the backend recording it. So a merchant who has only completed Step 1 via the partner API currently lands back on **Step 1** (pre-filled, since that data is already saved), not Step 2, when this redirect fires. This is the same behavior the existing admin "Resume Merchant Signup" button already produces for any deal at this stage — it isn't a gap introduced by this flow, and fixing it would mean changing EMAP's own step-routing logic, out of scope for a partner-side form.

---

## 2. Flow Option 3 (resume email) — `POST /v1/signup/resume-link`

Implemented on `epd-emap` (branch `feat/emap-signup-flow-2-3-backend`, off `origin/staging`), verified via live requests.

This is EMAP's existing `SignupController::sendFinishLaterLink()` (`POST /signup/finish-later-link`, `web` route) — the "finish later" feature already used on EMAP's own signup pages — copied as-is into an unauthenticated `v1` endpoint. The only real difference is the `auth()->check()` gate: that route identifies the merchant via `Auth::user()` (guaranteed present because of the gate), which isn't available to a partner-hosted form's visitor, so this endpoint resolves the merchant by the given `email` instead.

Unauthenticated. Callable from any step 1–6 of the partner-hosted form (e.g. a "Finish later" button).

**Request**:
```json
{ "email": "merchant@example.com" }
```

**Response** (200, always when `email` is valid — matches the original's behavior of never revealing whether an email matches an account):
```json
{ "status": true, "message": "Resume link sent" }
```

- `422` — `email` missing/invalid, e.g. `{ "status": false, "message": "The email field is required." }`.
- `429` — rate-limited (5 requests / 5 minutes per IP).

**How it's built**: validates `email` inline with `Validator::make()` (matching `sendFinishLaterLink`'s own style — no FormRequest class), looks up `User::where('email', $email)->first()` (replacing `Auth::user()`), then dispatches `TriggerFinishLaterEmail::dispatch($email, $url)` just like the original. The resume URL itself comes from a new `IndexController::resolveResumeUrlForUser($userId, $email)` method — a sibling of the existing `resolveResumeUrl()`, not a modification of it, since this endpoint has no authenticated user to default to and an unknown email leaves `$userId` null. `resolveResumeUrl()` itself is untouched. Rate limiting is by IP (the original's `handleAutoSaveRateLimit` keys by session id and is a `private` method on a different controller, so it can't be reused directly).

**Files touched** (only what this one endpoint needs):
- `app/Http/Controllers/API/SignupAPIController.php` — one new method, `resumeLink()`.
- `app/Http/Controllers/Client/IndexController.php` — one new method, `resolveResumeUrlForUser($userId, $emailForFallback)`, added alongside the existing `resolveResumeUrl()` (which is unchanged) so it can reuse the same private lookup helpers (`getUserCompanyIds`/`findIncompleteApplication`/`findAnyApplication`).
- `routes/api.php` — one new route, `POST /v1/signup/resume-link`.

No migration, no model changes — an earlier version of this work added a `purpose` column to `merchant_signup_tokens` for a custom auto-login-token approach to Option 2; that's been reverted in favor of reusing the existing `/upload-document` route above.

---

## Summary

| Flow | New backend work |
|---|---|
| Option 2 (redirect after Step 1) | None — reuses existing `GET /upload-document/{uuid}?redirect=1` |
| Option 3 (resume email) | `POST /v1/signup/resume-link` — `SignupController::sendFinishLaterLink` reused as-is, auth swapped for an email lookup |

**Not yet done**: this work is uncommitted on a local-only `epd-emap` branch (`feat/emap-signup-flow-2-3-backend`, based on `origin/staging`) — not committed, pushed, reviewed, or merged.
