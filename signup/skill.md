---
name: creating-emap-signup-forms
description: Build a complete merchant onboarding system with a comprehensive 6-step signup form. Collect account details, company information, product specifics, owner information, banking details, and final preferences. Features intelligent field dependencies, multi-owner support, and seamless API integration with production-ready validation and error handling.
---

# Merchant Signup Form Specification

Production-ready specification for building the complete 6-step signup form.

---

## Form Overview

| Step | Title | File | Fields | Features |
|------|-------|------|--------|----------|
| 1 | Account Information | [STEP1_ACCOUNT_INFORMATION.md](steps/STEP1_ACCOUNT_INFORMATION.md) | 10 | Phone formatting, country/state selection |
| 2 | Company Information | [STEP2_COMPANY_INFORMATION.md](steps/STEP2_COMPANY_INFORMATION.md) / [CONDITIONALS](steps/STEP2_COMPANY_INFORMATION_CONDITIONALS.md) | 26 | Manual address entry (legal + physical), revenue model with nested conditionals (country/state collected in Step 1) |
| 3 | Product Information | [STEP3_PRODUCT_INFORMATION.md](steps/STEP3_PRODUCT_INFORMATION.md) | 13 | Fulfillment details, transaction slider (multiples of 5) |
| 4 | Owner Information | [STEP4_OWNER_INFORMATION.md](steps/STEP4_OWNER_INFORMATION.md) / [FIELDS](steps/STEP4_OWNER_INFORMATION_FIELDS.md) / [IMPLEMENTATION](steps/STEP4_OWNER_INFORMATION_IMPLEMENTATION.md) | 45+ | Primary contact, Owner 1 & conditional Owner 2, manual address entry, driver's license, financial history |
| 5 | Banking Information | [STEP5_BANKING_INFORMATION.md](steps/STEP5_BANKING_INFORMATION.md) | 9 | Routing validation, country-specific fields |
| 6 | Final Details | [STEP6_FINAL_DETAILS.md](steps/STEP6_FINAL_DETAILS.md) | 11 | Interest selection, terms acceptance |

**Total Fields**: 114+ (10+26+13+45+9+11 across all 6 steps; Step 4 varies with conditional Owner 2)

---

## Quick Reference Files

**Supporting Documentation**:
- [specification.json](specification.json) - Machine-readable field definitions, validation rules, and dependency graph
- [reference/DEPENDENT_FIELDS_STEP1_3.md](reference/DEPENDENT_FIELDS_STEP1_3.md) / [STEP4_6](reference/DEPENDENT_FIELDS_STEP4_6.md) - Complete conditional field logic
- [reference/DROPDOWNS_REFERENCE.md](reference/DROPDOWNS_REFERENCE.md) / [STATIC](reference/DROPDOWNS_STATIC_REFERENCE.md) - All dropdown values (API + static)
- [reference/api-examples.md](reference/api-examples.md) - Copy-paste JavaScript & cURL code

---

## Global Implementation

### Required Implementation Parameters

Before building the form, configure these **required** parameters:

| Parameter | Type | Description |
|-----------|------|-------------|
| **base_url** | string | Base URL for all API endpoints |
| **api_key** | string | Authentication key for all API requests. Retrieve this from the **Partner Dashboard** (API/Developer settings) before starting implementation |

---

### Authentication Headers (CRITICAL)

**Form Submission (POST)**:
```javascript
headers: {
    'Content-Type': 'application/json',
    'X-API-Key': api_key,
    'Accept': 'application/json'
}
```

**Dropdown Data (GET)**:
```javascript
headers: {
    'Authorization': api_key,
    'Content-Type': 'application/json'
}
```

---

### Error Handling

**Status codes actually returned by the backend** (verified against `SignupAPIController` and its FormRequest classes):

| Code | When | Endpoints |
|------|------|-----------|
| 200 | Success — **every** success response uses 200, never 201 | All |
| 200 | ⚠️ "Company already exists" during Step 1 — `status:false` but HTTP 200 (no error code set) | Step 1 only |
| 400 | Bad request — generic exception caught | Steps 1, 2/3/5/6 |
| 403 | Blocked region (geo-check), or auth error | Step 1 (geo-block); auth middleware (all) |
| 404 | Application or Company not found for the given `uuid` | Steps 2/3/5/6, Step 4 |
| 422 | Validation failed | All (via FormRequest `failedValidation`) |
| 500 | Server error — generic exception caught | Step 4 only (steps 1/2/3/5/6 use 400 for the same case) |

⚠️ **Always check the `status` boolean in the response body — do not rely on the HTTP status code alone.** The Step 1 "Company already exists" case returns HTTP 200 with `status:false`.

**422 Validation Error** (all endpoints, thrown by each FormRequest's `failedValidation`):
```json
{
  "status": false,
  "message": "Validation failed",
  "errors": {
    "email": ["Email is required"],
    "country": ["Country must be a valid country ID"]
  }
}
```
Steps 2/3/5/6 additionally include `"step": <step_count>` in this response.

**404 Not Found** (Steps 2/3/4/5/6 — application or company missing for the `uuid`):
```json
{ "status": false, "message": "Application not found" }
```
or `"message": "Company not found"`.

**400 / 500 Generic Error** (uncaught exception):
```json
{ "status": false, "message": "Error", "data": "<exception message>" }
```
Step 4 (`handleOwnership`) uses HTTP 500 for this; every other step uses HTTP 400 for the same shape.

**403 Blocked Region** (Step 1 only, geo-IP check):
```json
{ "status": false, "message": "Unauthorised access." }
```

**Recommended client handling**:
```javascript
function handleStepResponse(response, httpStatus) {
  if (response.status === true) {
    // proceed to next step using response.uuid
    return;
  }
  if (httpStatus === 422) {
    // response.errors is an object: { field: [messages] }
    for (const [field, messages] of Object.entries(response.errors || {})) {
      displayErrorForField(field, messages[0]);
    }
    return; // stay on current step, do not change URL
  }
  if (httpStatus === 404) {
    // uuid is stale/invalid — restart from Step 1
    return;
  }
  // 400/403/500 or status:false with HTTP 200 — show response.message to the user
  showError(response.message);
}
```

---

## Guidance

### How Step Progression Works

1. Submitting Step 1 returns a `uuid` in the response — this is the unique identifier for the signup session.
2. Pass that same `uuid` in the payload of every subsequent step submission (Steps 2–6) so each step's data is saved against the correct session.
3. The `uuid` does not change for the lifetime of the signup flow — obtain it once, then reuse it on every step save.

### Page Refresh Behavior

- If the user refreshes the page mid-flow, the form must resume on the same step they were on — it should **not** reset back to Step 1.
- Persist the current step number and `uuid` so they can be restored after a refresh, using whichever state mechanism fits the implementation (e.g., URL, storage, session).

### Field Format Rules

| Field Type | Rule |
|------------|------|
| Date fields | Submit in `Y-m-d` format (e.g., `2026-07-30`) |
| Phone fields | Must be a valid, correctly formatted phone number before submission |
| Currency / amount fields | Always submit as an integer — no decimals, symbols, or commas |

### Keep the Selected Country From Step 1

`country` is collected **once**, on Step 1, and is never re-collected or shown as a dropdown on any later step. Steps 2–5 do not render a `#country` `<select>` — they read the persisted Step 1 value and use it to decide which dependent fields to show or hide.

- **Persist the Step 1 `country` value** (server-side on the application/company record, and/or client-side alongside the `uuid`) so every later step can read it without re-asking the user.
- **Do not bind a `change` handler to a `#country` element on Steps 2–5** — there isn't one. Evaluate the persisted value once when the step loads instead.
- Use the **country code/slug** (`"US"`, `"CA"`, `"MX"`, ...) for every comparison — never the numeric `id`.

Fields that depend on the persisted Step 1 country:

| Step | Field(s) | Behavior |
|------|----------|----------|
| 2 | `business_register_number` | Shown/required only if `country ≠ "US"` |
| 2 | `federal_tax_id` | Label and input mask vary by `country` |
| 4 | `driver_license_state`, `driver_license_expiration_date` | Shown/required only if the **owner's** `country = "US"` (this is the owner's own address country from Step 4, not the Step 1 company country) |
| 5 | `institution_number`, `customer_pay_currency` | Shown/required only if `country = "CA"` (Canada) |
| 5 | `routing_number`, `account_number` | Label/format vary by `country` |

---

## Testing Checklist

**Per-step**: dropdown data loads, required-field validation fires, conditional fields toggle correctly, submission succeeds and returns a `uuid`.

**Must pass before considering the form done**:
- [ ] Step 1 → Step 6 completes end-to-end with a single persisted `uuid`
- [ ] Physical address fields (Step 2) only required when `is_physical_address_same_as_legal_address=0`
- [ ] `business_state` (Step 1) appears only when `country="US"`
- [ ] Revenue model → subscription frequency → subscription frequency "Other" nested conditional (Step 2) works at both levels
- [ ] Transaction entry slider (Step 3) only allows multiples of 5 and always sums to 100
- [ ] Owner 2 section (Step 4) appears only when `ownership_percentage[1] < 51`
- [ ] Owner 1 `first_name[1]`/`last_name[1]`/`email[1]` are omitted from submission when `primary_contact=1` (see [steps/STEP4_OWNER_INFORMATION.md](steps/STEP4_OWNER_INFORMATION.md))
- [ ] Owner `license` (Step 4) is always required, all countries; `driver_license_state`/`driver_license_expiration_date` required only when owner `country="US"`, hidden otherwise
- [ ] Page refresh resumes on the current step (not reset to Step 1)
- [ ] 422 responses display field errors without changing step/URL
