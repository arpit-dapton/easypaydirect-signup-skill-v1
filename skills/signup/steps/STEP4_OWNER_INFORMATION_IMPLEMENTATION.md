---
name: signup-step-4-implementation
description: Step 4 Owner Information - Libraries, JavaScript form submission, conditional logic, and testing checklist
---

# Step 4: Owner Information — Implementation & Testing

Continuation of [STEP4_OWNER_INFORMATION.md](STEP4_OWNER_INFORMATION.md) and [STEP4_OWNER_INFORMATION_FIELDS.md](STEP4_OWNER_INFORMATION_FIELDS.md). This file covers libraries, JavaScript form submission, conditional logic, data storage, and the testing checklist.

---

## Libraries Required

- Date picker library (e.g., Flatpickr, Bootstrap DatePicker, or similar) - For dob, driver_license_expiration_date, bankruptcy_discharged_date
- `intl-tel-input` - Phone formatting. Use the exact CDN URLs and version pinned in [STEP1_ACCOUNT_INFORMATION.md § Libraries Required](STEP1_ACCOUNT_INFORMATION.md#libraries-required) for every phone field on this step too — don't guess a different path/version.

---

## JavaScript

### Form Submission

⚠️ **CRITICAL: MUST send as FormData (application/x-www-form-urlencoded), NOT JSON**

All owner fields must use bracket array notation `[1]` or `[2]`, NOT object notation `{1: value}`

This single handler covers both the **first submission** and any **re-submission** (user navigated back and submitted again). Do not write two separate handlers.

```javascript
$('#ownership-form').on('submit', function(e) {
    e.preventDefault();

    const uuid = getSignupUuid();  // however the implementation stores it

    // Step 1: Collect enabled fields via FormData
    // ✅ CORRECT: Use FormData — NOT JSON.stringify() or contentType: 'application/json'
    const formData = new FormData(this);

    // Step 2: Append values from disabled fields.
    // Disabled inputs are excluded from FormData by the browser. After Step 4 has been
    // submitted once, all fields are disabled (field lock). Without this block, a
    // re-submission would send an empty payload and fail validation.
    // Exclude hidden inputs — they are never disabled and would be double-appended.
    $(this).find('input:disabled:not([type="hidden"]), select:disabled, textarea:disabled')
        .each(function() {
            const name = $(this).attr('name');
            if (!name) return;
            const type = $(this).attr('type');
            if (type === 'radio' || type === 'checkbox') {
                if ($(this).is(':checked')) formData.append(name, $(this).val());
            } else {
                formData.append(name, $(this).val() || '');
            }
        });

    // Step 3: Append scalar fields not covered by the form's own inputs
    const primaryContact = $('input[name="primary_contact"]:checked').val();
    formData.append('uuid', uuid);
    formData.append('primary_contact', primaryContact);
    formData.append('primary_contact_job_title', $('#primary_contact_job_title').val() || '');
    formData.append('step_count', 4);

    // Step 4: If primary_contact=1, remove first_name[1]/last_name[1]/email[1] entirely —
    // these fields are not rendered when primary_contact=1, so they won't be in FormData
    // from steps 1 or 2 above. This delete is a safety guard only.
    if (primaryContact === '1') {
        formData.delete('first_name[1]');
        formData.delete('last_name[1]');
        formData.delete('email[1]');
    }

    $.ajax({
        url: '/api/v1/ownership',
        type: 'POST',
        data: formData,
        processData: false,        // ← Critical: tells jQuery to NOT process FormData
        contentType: false,        // ← Critical: lets browser set multipart/form-data
        // ❌ NEVER: contentType: 'application/json'
        // ❌ NEVER: JSON.stringify(formData)

        success: function(response) {
            if (response.status) {
                saveStepData(4, '#ownership-form'); // persist values for back-navigation
                disableStep4Fields();               // lock all fields after successful save
                goToStep(5, uuid);                  // however the implementation navigates
            }
        },
        error: function(xhr) {
            const errors = xhr.responseJSON?.errors || {};
            for (const [field, messages] of Object.entries(errors)) {
                alert(field + ': ' + messages[0]);
            }
        }
    });
});
```

**What Goes Wrong if You Use JSON**:
```javascript
// ❌ WRONG - Sends as JSON
$.ajax({
    url: '/api/v1/ownership',
    type: 'POST',
    contentType: 'application/json',     // ❌ WRONG
    data: JSON.stringify(formData),      // ❌ WRONG
    ...
});

Result: Laravel receives literal key "first_name[1]" as string, not array
Validation sees: {first_name[1]: "value"} ← No array structure
Error: "first_name.1 is required" (expected array, got nothing)
```

**Correct FormData Request**:
```
POST /api/v1/ownership HTTP/1.1
Content-Type: multipart/form-data; boundary=...

--boundary
Content-Disposition: form-data; name="first_name[1]"

John
--boundary
Content-Disposition: form-data; name="last_name[1]"

Doe
--boundary
...
```

Laravel receives and parses as: `first_name => [1 => "John"], last_name => [1 => "Doe"]`
Validation matches: `first_name.1`, `last_name.1` ✅

### Important: Do NOT Fetch User Data

⚠️ **When `primary_contact=1`, do NOT call any API to fetch user data.**
- The fields `first_name[1]`, `last_name[1]`, `email[1]` should be **completely hidden** from the user
- These fields should **not be submitted** to the API
- The backend uses the user's Step 1 data (first_name, last_name, email from the User record)
- No additional API calls needed

---

## Conditional Logic

### Primary Contact Fields
```
IF primary_contact = 0 (NOT owner):
  SHOW: primary_contact_job_title (REQUIRED)
  SHOW: first_name[1], last_name[1], email[1] (EDITABLE form fields - REQUIRED)
  SUBMIT: first_name[1], last_name[1], email[1] to API (from editable fields)

ELSE (primary_contact = 1, IS owner):
  HIDE: primary_contact_job_title
  HIDE: first_name[1], last_name[1], email[1] fields — do not render them
  OMIT: first_name[1], last_name[1], email[1] from the submitted payload entirely
  Backend automatically uses the Step 1 user's data for Owner 1 — no frontend action needed
```

**Key Points**:
- When primary_contact=1: No pre-fill, no fetch, no display box — just don't render or submit these three fields
- When primary_contact=0: Show editable form fields for owner data, required
- Form submission removes first_name[1]/last_name[1]/email[1] from the FormData when primary_contact=1

### Owner 2 Section
```
IF ownership_percentage[1] < 51 → SHOW
ELSE → HIDE & CLEAR
```

### Driver License Fields

⚠️ **`license[1]`/`license[2]` are always visible and required, for every country — they are NOT part of this conditional.** Only `driver_license_state` and `driver_license_expiration_date` toggle:

```
IF country[1] = "US" → SHOW driver_license_state, driver_license_expiration_date (REQUIRED)
ELSE → HIDE & CLEAR driver_license_state, driver_license_expiration_date

license[1] is ALWAYS shown and required, regardless of country
```

### Bankruptcy Cascade
```
IF bankruptcy_filed[1] = 1 → SHOW bankruptcy_discharged
  IF bankruptcy_discharged[1] = 1 → SHOW bankruptcy_discharged_date
```

---

## Data Storage

**Models**:
- `Application`: user_id, primary_contact_user_id, primary_contact_job_title
- `Company`: user_id, primary_contact_user_id, primary_contact_job_title
- `CompanyOwner`: All owner data (21+ fields per owner)

**User Creation**:
- Owner 1: New user created with form data
- Owner 2: New user created if data provided

---

## Full Field Lock After Submission

**Rule**: Once Step 4 has been successfully submitted, ALL Step 4 fields must be disabled. Fields must remain fully editable until the moment of a successful submission — never before.

⚠️ **Do NOT disable any fields on page/component load unless Step 4 has already been submitted.** Disabling fields unconditionally (e.g. in `$(document).ready` without a submission check) is the most common implementation mistake and breaks the form for users who have not yet submitted Step 4.

### When to apply the lock

There are **two** moments when the lock must be applied:

1. **Immediately after a successful submission** — in the AJAX `success` callback, before navigating to Step 5.
2. **On page/component load when the user returns to Step 4** — check whether Step 4 was already submitted. Only apply the lock if it was.

### Implementation

#### 1. Apply lock on successful submission (in the AJAX success callback)

```javascript
success: function(response) {
    if (response.status) {
        // Step 4 submitted successfully — disable ALL fields before moving on
        disableStep4Fields();
        goToStep(5, uuid); // however the implementation navigates
    }
}
```

#### 2. Apply lock on page load if Step 4 was already submitted (returning user)

Order matters: **restore saved values first, then apply the lock.** Disabling before restoring leaves all fields disabled AND empty.

```javascript
$(document).ready(function() {
    // 1. Restore previously saved field values so the user sees their data
    restoreStepData(4, '#ownership-form');

    // 2. Re-trigger conditional visibility based on restored values
    $('input[name="primary_contact"]:checked').trigger('change');
    $('input[name="ownership_percentage[1]"]').trigger('input');
    $('select[name="country[1]"]').trigger('change');
    // (add any other Step 4 conditionals here)

    // 3. Only after data is restored, apply the field lock if Step 4 was already submitted
    const stepCount = getSignupStepCount(); // however the implementation retrieves it
    if (stepCount >= 4) {
        // Step 4 was already submitted — disable all visible fields
        disableStep4Fields();
    }
    // If stepCount < 4, Step 4 has NOT been submitted yet — leave all fields editable
});
```

#### The disable helper

```javascript
function disableStep4Fields() {
    // Disable visible interactive fields only.
    // ⚠️ Exclude [type="hidden"] — hidden inputs must never be disabled.
    //    Disabled hidden inputs are excluded from FormData, which would silently drop
    //    uuid, step_count, and any other hidden values from re-submission payloads.
    // NOTE: the submit button is intentionally NOT disabled — re-submission must remain possible.
    //    See skill.md → "Re-submission Behavior".
    $('#ownership-form input:not([type="hidden"]), #ownership-form select, #ownership-form textarea')
        .prop('disabled', true);
}
```

### Interaction with existing conditional logic

- Because `primary_contact` is disabled after submission, its `change` handler will never fire on a returning user. Set the Owner 1 name/email section visibility once on load based on the stored/pre-filled value — do not rely on the change event.
- Because `ownership_percentage[1]` is disabled, the Owner 2 visibility toggle will not fire on a returning user. Initialize Owner 2 visibility on load from the stored value.

---

## Testing Checklist

- [ ] primary_contact=1: Hides first_name/last_name/email fields, does NOT submit them, uses Step 1 user data
- [ ] primary_contact=0: Shows job_title, first_name/last_name/email fields, user enters owner data
- [ ] ownership < 51%: Shows Owner 2 section
- [ ] ownership ≥ 51%: Hides Owner 2 section
- [ ] `license` field is always visible/required for every country (not conditional)
- [ ] country=US: Shows `driver_license_state` and `driver_license_expiration_date`
- [ ] country≠US: Hides `driver_license_state` and `driver_license_expiration_date`
- [ ] bankruptcy_filed=1: Shows discharged field
- [ ] bankruptcy_discharged=1: Shows date field
- [ ] DOB date picker blocks any date less than 18 years ago (minimum age enforcement, both Owner 1 and Owner 2)
- [ ] All required fields validated
- [ ] Proceeds to Step 5 on success (uuid carried forward)
- [ ] Shows errors on validation failure
- [ ] All Step 4 fields are **editable** before the form is submitted (no premature disabling)
- [ ] All Step 4 fields become **disabled** immediately after a successful Step 4 submission
- [ ] All Step 4 fields are **disabled** when a returning user lands on Step 4 after it was already submitted (stepCount ≥ 4)
- [ ] The submit button remains **enabled** after Step 4 has been submitted (re-submission must be possible)
- [ ] Re-submitting Step 4 (with disabled fields) sends a complete payload — disabled field values are appended manually to FormData

---

**See also**: [STEP4_OWNER_INFORMATION.md](STEP4_OWNER_INFORMATION.md) and [STEP4_OWNER_INFORMATION_FIELDS.md](STEP4_OWNER_INFORMATION_FIELDS.md).
