---
name: signup-step-1
description: Step 1 Account Information - First step of merchant signup with basic details and phone formatting
---

# STEP 1: Account Information

First step of the 6-step signup form. Collects basic merchant contact and company information.

---

## Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| first_name | text | Yes | UI cap: max 60 chars. (Backend actually allows up to 255 — 60 is a recommended UI limit, not a validation ceiling.) |
| last_name | text | Yes | UI cap: max 60 chars. Same note as first_name. |
| email | email | Yes | Valid email format. **Must also be unique** — backend rejects with 422 ("This email address is already registered") if the email already exists on a `users` row. Handle this case distinctly in the error UI (e.g. suggest logging in) rather than showing a generic validation message. |
| phone | tel | Yes | Use intl-tel-input library |
| name | text | Yes | Company name. UI cap: max 60 chars (backend allows up to 255 — same note as first_name). |
| website | url | Yes | Backend regex is domain-only with an optional single trailing slash — it does **not** accept a path (`https://facebook.com/mybusiness` fails). Despite the "or social media profile" framing below, only a bare social-media domain (no page/profile path) currently validates. Flag this with product before promising path-based social profile URLs work. |
| country | select | Yes | Normal dropdown (no search) - API: `/api/partner/countries` |
| business_state | select | Yes | Normal dropdown (no search) - Show only if country="US", API: `/api/partner/states` |
| annual_sales | number | Yes | Numeric only, no currency symbols or commas |
| promo_code | text | No | Optional referral code |
| partner_key | text | No | Optional partner API key for attribution — see skill.md → "MANDATORY FIRST STEP" (top of file). Ask the implementer if they have one before including it; omit entirely if not |
| step_count | — | **Yes** | Not a form field — always send the literal value `1` in the payload (not user-editable). Without it, the backend never records the application as having reached step 1, which breaks any later resume/redirect back into EasyPayDirect (it lands on Step 1 again instead of Step 2, even though Step 1 data is already saved). |

---

## Country Dropdown

**Field**: `country`  
**Type**: SELECT dropdown  
**API Endpoint**: `GET /api/partner/countries`

**Response Format**:
```json
{
  "success": true,
  "data": [
    { "name": "United States", "code": "US", "id": "1" },
    { "name": "Canada", "code": "CA", "id": "2" },
    { "name": "Mexico", "code": "MX", "id": "3" },
    ...
  ]
}
```

**⚠️ CRITICAL: Use `code` (slug) as option value, NOT `id`**
```html
<!-- CORRECT -->
<option value="US">United States</option>
<option value="CA">Canada</option>
<option value="MX">Mexico</option>

<!-- WRONG - do NOT use id -->
<option value="1">United States</option>
<option value="2">Canada</option>
```

**Form Submission**: Submit the `code` value (e.g., "US", "CA", "MX")

---

## Libraries Required

- `intl-tel-input@22.0.2` - Phone formatting with country selector
- `dependent-fields.js` - Conditional field visibility

**Use these exact CDN URLs — do not guess or substitute a different CDN/path/version.** `intl-tel-input`'s build output path has changed across major versions, so a guessed path (e.g. `js/utils.js` instead of `build/js/utils.js`, or an unpkg URL instead of jsdelivr) is a common source of a silently-broken phone field. Verified working (HTTP 200) as of this writing:
```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/intl-tel-input@22.0.2/build/css/intlTelInput.css">
<script src="https://cdn.jsdelivr.net/npm/intl-tel-input@22.0.2/build/js/intlTelInput.min.js"></script>
```
```js
window.intlTelInput(phoneInputEl, {
  utilsScript: 'https://cdn.jsdelivr.net/npm/intl-tel-input@22.0.2/build/js/utils.js',
  initialCountry: 'us'
});
```

---

## API Integration

**Dropdown Data**:
```
GET /api/partner/countries
GET /api/partner/states
```

**Form Submission**:
```
POST /api/v1/signup
Headers: None required.
Payload: 
  country: "US" (use code/slug, not id)
  all other Step 1 fields
  step_count: 1
  partner_key: "..." (OPTIONAL — only include if the implementer has a partner
                API key; see skill.md → "MANDATORY FIRST STEP" (top of file). This is
                not authentication — signup succeeds even if omitted or invalid.)
Response: { uuid, step_count, message }
```

---

## Validation

- First/Last name: required, UI cap 60 chars (backend allows up to 255)
- Email: required, valid format, must be unique (422 if already registered)
- Phone: required, valid number (intl-tel-input validates)
- Website: required, valid URL — domain only, no path (see Fields table note)
- Country: required, must be valid country code (e.g., "US", "CA")
- Business State: required if country="US" (use code, not id)
- Annual Sales: required, numeric only (no currency symbols or commas)

---

## Dependent Fields

- **business_state**: Show if country="US" (hidden by default)

---

## Form Submission & Redirect

**On Success (HTTP 200/201)**:
```javascript
// Response example:
{
  "status": true,
  "message": "Account created successfully",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "step_count": 1
}

// 1. Persist uuid and country for all subsequent steps
localStorage.setItem('signup_uuid', response.uuid);
localStorage.setItem('signup_step', 1);
localStorage.setItem('signup_country', $('#country').val()); // code/slug, e.g. "US"

// 2. Save Step 1 field values so they are restored if the user navigates back
saveStepData(1, '#step1Form'); // see skill.md → Form Data Persistence

// 3. Proceed to Step 2
```

**On Validation Error (HTTP 422)**: standard shape — see skill.md → Error Handling. Stay on Step 1, display field errors, do not change URL.

---

## Step Lock After Submission

**Rule**: Once Step 1 has been successfully submitted, if the user navigates back to Step 1, ALL fields must be disabled. The user cannot edit or resubmit Step 1.

### When to apply the lock

On Step 1 page/component load, check whether a `uuid` already exists in the persisted signup state (localStorage, sessionStorage, URL, or wherever the implementation stores it). If a `uuid` is present, Step 1 was already submitted — apply the lock immediately before rendering.

### What to disable

- Every `<input>`, `<select>`, and `<textarea>` inside the Step 1 form
- The submit button

⚠️ **Do not call `iti.setDisabled(true)`.** `intl-tel-input` has no `setDisabled` method in any version through at least `22.0.2` (the version pinned above) — it is not part of the library's public API. Calling it throws `TypeError: iti.setDisabled is not a function`. Because this is easy to trigger inside a `.then()` success handler right after a real, successful Step 1 submission, the thrown error commonly gets caught by an unrelated outer `.catch()` meant for genuine network failures and misreported to the user as a network error — even though Step 1 actually succeeded and a `uuid` was returned. The phone `<input>` that `intl-tel-input` wraps is disabled automatically by the plain `.prop('disabled', true)` / `disabled = true` call on "every `<input>`... inside the Step 1 form" above — no separate widget-specific call is needed or exists.

### Implementation

```javascript
$(document).ready(function() {
    const uuid = getSignupUuid(); // however the implementation retrieves it

    if (uuid) {
        // 1. Restore saved field values so the user sees their data (not empty fields)
        //    Must run BEFORE disabling so values are written while fields are still writable
        restoreStepData(1, '#step1Form'); // see skill.md → Form Data Persistence

        // 2. Re-apply conditional field visibility based on restored country value
        //    (e.g. business_state is shown only when country="US")
        $('#country').trigger('change');

        // 3. Re-set the intl-tel-input phone value from restored data if needed
        //    (iti.setNumber() accepts the E.164 number stored during submission)
        const saved = JSON.parse(localStorage.getItem('signup_step_1_data') || '{}');
        if (typeof iti !== 'undefined' && saved.phone) {
            iti.setNumber(saved.phone);
        }

        // 4. Lock the entire form — fields are now readable but not editable.
        //    This also disables the plain <input> that intl-tel-input wraps —
        //    do NOT also call iti.setDisabled(true); that method does not
        //    exist on this library and throws if called (see "What to
        //    disable" above).
        $('#step1Form input, #step1Form select, #step1Form textarea')
            .prop('disabled', true);
        $('#step1Form button[type="submit"]').prop('disabled', true);

        // Optional: show a read-only notice to the user
        // e.g. $('#step1Notice').text('This step has already been submitted.').show();
    }
});
```

### Why

The `uuid` is created by the backend at Step 1 submission and identifies the active signup session. Its presence is the authoritative signal that Step 1 is complete. Resubmitting Step 1 with the same email would be rejected with a 422 "email already registered" error, so the form must prevent it client-side rather than showing a confusing error.

---

## Field Summary

**Total Fields**: 11  
**Required Fields**: 9 (all except promo_code and partner_key)  
**Conditionally Required**: 1 (business_state - required if country=US)  
**Optional**: 2 (promo_code, partner_key)  
**Phone Fields**: 1 (intl-tel-input)
