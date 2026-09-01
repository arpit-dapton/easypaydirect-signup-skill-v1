---
name: signup-backend-dependencies
description: EMAP (epd-emap) backend endpoints powering Flow Option 2 (redirect) and Option 3 (resume email)
---

# Backend Endpoints — Flow Options 2 & 3

Status as of **2026-09-01**, verified against `origin/staging` of `epd-emap`. Backend work is on branch `feat/emap-signup-flow-2-3-backend`, pushed to `origin`, not yet merged — confirm it's deployed to your target environment before relying on it.

---

## 1. Flow Option 2 (Step 1 only, redirect to EMAP)

Reuses an existing EMAP route — no new backend endpoint needed. After Step 1 succeeds (`POST /v1/signup` returns `uuid`), redirect the browser to:

```
GET {EMAP_APP_URL}/upload-document/{uuid}?redirect=1
```

Unauthenticated — logs the merchant in by `uuid` alone and routes them to wherever their application currently stands (back into the form if incomplete, to signing if a template applies, or to the dashboard if already signed).

**Requires `"step_count": 1` in the Step 1 payload** — without it, this redirect lands back on Step 1 instead of Step 2. See [steps/STEP1_ACCOUNT_INFORMATION.md](../steps/STEP1_ACCOUNT_INFORMATION.md).

---

## 2. Flow Option 3 (resume email) — `POST /v1/signup/resume-link`

Unauthenticated. Callable from Step 2 onward of the partner-hosted form (e.g. a "Finish later" button) — not on Step 1, since no `uuid`/application exists yet to resume into.

**Request**:
```json
{ "email": "merchant@example.com" }
```

**Response** (200, always when `email` is valid — never reveals whether an email matches an account):
```json
{ "status": true, "message": "Resume link sent" }
```

- `422` — `email` missing/invalid, e.g. `{ "status": false, "message": "The email field is required." }`.
- `429` — rate-limited (5 requests / 5 minutes per IP).

---

## Summary

| Flow | Endpoint |
|---|---|
| Option 2 (redirect after Step 1) | `GET {EMAP_APP_URL}/upload-document/{uuid}?redirect=1` — requires `"step_count": 1` in the Step 1 payload |
| Option 3 (resume email) | `POST /v1/signup/resume-link` — only enable from Step 2 onward |
