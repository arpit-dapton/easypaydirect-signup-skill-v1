---
name: signup-step-4-implementation
description: Step 4 Owner Information - Libraries, JavaScript form submission, conditional logic, and testing checklist
---

# Step 4: Owner Information — Implementation & Testing

Continuation of [STEP4_OWNER_INFORMATION.md](STEP4_OWNER_INFORMATION.md) and [STEP4_OWNER_INFORMATION_FIELDS.md](STEP4_OWNER_INFORMATION_FIELDS.md). This file covers libraries, JavaScript form submission, conditional logic, data storage, and the testing checklist.

---

## Libraries Required

- Date picker library (e.g., Flatpickr, Bootstrap DatePicker, or similar) - For dob, driver_license_expiration_date, bankruptcy_discharged_date
- `intl-tel-input` - Phone formatting

---

## JavaScript

### Form Submission

⚠️ **CRITICAL: MUST send as FormData (application/x-www-form-urlencoded), NOT JSON**

All owner fields must use bracket array notation `[1]` or `[2]`, NOT object notation `{1: value}`

**Correct Implementation**:
```javascript
$('#ownership-form').on('submit', function(e) {
    e.preventDefault();

    const uuid = getSignupUuid();  // however the implementation stores it

    // ✅ CORRECT: Use FormData to send as application/x-www-form-urlencoded
    const formData = new FormData(this);
    const primaryContact = $('input[name="primary_contact"]:checked').val();

    // ❌ DO NOT: Use JSON.stringify() or contentType: 'application/json'

    formData.append('uuid', uuid);
    formData.append('primary_contact', primaryContact);
    formData.append('primary_contact_job_title', $('#primary_contact_job_title').val() || '');
    formData.append('step_count', 4);

    // If primary_contact=1, REMOVE first_name[1], last_name[1], email[1] from submission
    if (primaryContact === '1') {
        formData.delete('first_name[1]');
        formData.delete('last_name[1]');
        formData.delete('email[1]');
    }

    $.ajax({
        url: '/api/v1/ownership',
        type: 'POST',
        // No auth header required
        data: formData,
        processData: false,        // ← Critical: tells jQuery to NOT process FormData
        contentType: false,        // ← Critical: lets browser set multipart/form-data
        // ❌ NEVER: contentType: 'application/json'
        // ❌ NEVER: JSON.stringify(formData)

        success: function(response) {
            if (response.status) {
                goToStep(5, uuid); // however the implementation navigates
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

**Correct FormData Request** (no auth header required):
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
- [ ] All required fields validated
- [ ] Proceeds to Step 5 on success (uuid carried forward)
- [ ] Shows errors on validation failure

---

**See also**: [STEP4_OWNER_INFORMATION.md](STEP4_OWNER_INFORMATION.md) and [STEP4_OWNER_INFORMATION_FIELDS.md](STEP4_OWNER_INFORMATION_FIELDS.md).
