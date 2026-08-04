---
name: signup-step-4
description: Step 4 Owner Information - Field naming format, FormData requirement, date pickers, primary contact, and owner data handling
---

# Step 4: Owner Information

Complete specification for collecting business owner information including primary contact status, owner details, addresses, and financial history.

**Continue to**: [STEP4_OWNER_INFORMATION_FIELDS.md](STEP4_OWNER_INFORMATION_FIELDS.md) (Owner 1/2 fields, validation, API responses) → [STEP4_OWNER_INFORMATION_IMPLEMENTATION.md](STEP4_OWNER_INFORMATION_IMPLEMENTATION.md) (libraries, JavaScript, conditional logic, testing).

---

## ⚠️ CRITICAL: Field Naming Format

**ALL owner fields MUST use bracket array notation consistently:**

```
✅ CORRECT FORMAT (all use [1] and [2]):
- first_name[1], last_name[1], email[1], phone[1]
- bankruptcy_filed[1], bankruptcy_discharged[1], bankruptcy_discharged_date[1]
- city[1], country[1], state[1], postal_code[1]
- All fields follow: fieldname[1] or fieldname[2]

❌ WRONG FORMAT (mixed object and bracket notation):
- bankruptcy_discharged_date: {1: "2008-05-23", 2: ""}  ← DO NOT USE
- city: {1: "RJ", 2: ""}  ← DO NOT USE
- bankruptcy_filed[1]  ← Inconsistent with above
```

Form generators and JavaScript must ensure ALL fields use bracket notation `[1]` or `[2]`, never object `{1: value, 2: value}` format.

---

## Overview

Step 4 collects:
- Primary contact status (is the user the owner?)
- Owner 1 information (21 fields)
- Owner 2 information (22 fields, conditional if ownership < 51%)
- Owner address (manual entry, country as normal dropdown)
- Multi-level conditional field visibility

**Total Fields**: 45+

---

## ⚠️ CRITICAL: FormData Requirement

**BEFORE implementing, read this carefully:**

```javascript
// ✅ CORRECT - Use FormData
const formData = new FormData(form);
formData.append('first_name[1]', value);

$.ajax({
    data: formData,
    processData: false,      // ← REQUIRED
    contentType: false,      // ← REQUIRED
});

// ❌ COMPLETELY WRONG - Do NOT use JSON
const data = { 'first_name[1]': value };  // ❌
JSON.stringify(data)                       // ❌
contentType: 'application/json'            // ❌

// WHY IT MATTERS:
// FormData sends as: first_name[1]=John → Laravel parses as array → Validation works ✅
// JSON sends as: {"first_name[1]":"John"} → Laravel sees literal key → Validation fails ❌
```

---

## API Endpoint

**POST `/api/v1/ownership`**

**Authentication**: X-API-Key header (from Step 1)

**Content-Type**: application/x-www-form-urlencoded (FormData automatically sets this)

---

## Date Picker Fields

Step 4 has three date fields for each owner (Owner 1 and Owner 2). All must use date picker with format YYYY-MM-DD.

### 1. Date of Birth (dob)

**Field**: `dob[1]`, `dob[2]`
**Type**: DATE input with picker
**Format**: YYYY-MM-DD
**Required**: Yes
**Validation**: Must be a valid past date (cannot be today or future)

**HTML**:
```html
<div class="form-group">
    <label for="dob_1">Date of Birth (Owner 1)</label>
    <input type="date" id="dob_1" name="dob[1]" class="form-control" required>
</div>
```

**JavaScript (Flatpickr)**:
```javascript
flatpickr('#dob_1', {
    mode: 'single',
    format: 'Y-m-d',
    maxDate: new Date(), // Cannot be today or future
    placeholder: 'YYYY-MM-DD'
});
```

### 2. Driver License Expiration Date

**Field**: `driver_license_expiration_date[1]`, `driver_license_expiration_date[2]`
**Type**: DATE input with picker
**Format**: YYYY-MM-DD
**Required**: Yes (only if country=US)
**Validation**: Must be today or in the future (cannot be past date)

**HTML**:
```html
<div class="form-group">
    <label for="driver_license_expiration_date_1">Driver License Expiration (Owner 1)</label>
    <input type="date" id="driver_license_expiration_date_1"
           name="driver_license_expiration_date[1]" class="form-control">
</div>
```

**JavaScript (Flatpickr)**:
```javascript
flatpickr('#driver_license_expiration_date_1', {
    mode: 'single',
    format: 'Y-m-d',
    minDate: new Date(), // Cannot be in the past
    placeholder: 'YYYY-MM-DD'
});
```

### 3. Bankruptcy Discharge Date

**Field**: `bankruptcy_discharged_date[1]`, `bankruptcy_discharged_date[2]`
**Type**: DATE input with picker
**Format**: YYYY-MM-DD
**Required**: Conditional (only if bankruptcy_discharged=1)
**Validation**: Must be a valid past date (cannot be today or future)

**HTML**:
```html
<div class="form-group" id="bankruptcy_discharged_date_wrapper" style="display: none;">
    <label for="bankruptcy_discharged_date_1">Bankruptcy Discharge Date (Owner 1)</label>
    <input type="date" id="bankruptcy_discharged_date_1"
           name="bankruptcy_discharged_date[1]" class="form-control">
</div>
```

**Show/Hide Logic**:
```javascript
// Show this field only when bankruptcy_discharged[1] = 1
$('input[name="bankruptcy_discharged[1]"]').on('change', function() {
    if ($(this).val() === '1') {
        $('#bankruptcy_discharged_date_wrapper').show();
    } else {
        $('#bankruptcy_discharged_date_wrapper').hide().val('');
    }
});
```

**JavaScript (Flatpickr)**:
```javascript
flatpickr('#bankruptcy_discharged_date_1', {
    mode: 'single',
    format: 'Y-m-d',
    maxDate: new Date(), // Cannot be today or future
    placeholder: 'YYYY-MM-DD'
});
```

---

## Primary Contact Field

**Question**: "Are you the business owner?"

| Value | Meaning |
|-------|---------|
| `1` | YES - User is the business owner |
| `0` | NO - User is not the business owner |

### Impact on Form

**If primary_contact=1 (IS owner)**:
- Hide `primary_contact_job_title` field
- **HIDE** `first_name[1]`, `last_name[1]`, `email[1]` fields — do not render them at all
- **DO NOT SUBMIT** these three fields to the API (omit them from the request entirely)
- **No pre-fill and no fetch needed** — the backend automatically uses the Step 1 user's `first_name`/`last_name`/`email` for Owner 1 when `primary_contact=1`. The frontend does not need to look this data up or display it back to the user.
- Set `primary_contact_user_id` = newly created Owner 1 user ID

**If primary_contact=0 (NOT owner)**:
- Show `primary_contact_job_title` field (REQUIRED)
- **SHOW** `first_name[1]`, `last_name[1]`, `email[1]` fields (user editable, REQUIRED)
- User manually enters Owner 1 info
- Keep `primary_contact_user_id` unchanged from Step 1

---

## Owner 1 Name/Email Fields — Show Only When Owner ≠ Primary Contact

⚠️ **Only pass `first_name[1]`, `last_name[1]`, `email[1]` when the owner is a different person from the primary contact (`primary_contact=0`).** When `primary_contact=1`, simply omit these three fields — the backend fills them in from the Step 1 user record automatically. No pre-fill, no display box, no extra API call.

### HTML

```html
<!-- Owner 1 name/email — only rendered/required when primary_contact=0 -->
<div id="owner_info_form" style="display: none;">
    <div class="form-group">
        <label for="first_name_1">First Name *</label>
        <input type="text" id="first_name_1" name="first_name[1]"
               class="form-control" required>
    </div>
    <div class="form-group">
        <label for="last_name_1">Last Name *</label>
        <input type="text" id="last_name_1" name="last_name[1]"
               class="form-control" required>
    </div>
    <div class="form-group">
        <label for="email_1">Email *</label>
        <input type="email" id="email_1" name="email[1]"
               class="form-control" required>
    </div>
</div>
```

### JavaScript - Toggle Visibility

```javascript
$(document).ready(function() {
    // Toggle Owner 1 name/email fields based on primary_contact selection
    $('input[name="primary_contact"]').on('change', function() {
        const isPrimaryContact = $(this).val() === '1';

        if (isPrimaryContact) {
            // Owner IS the primary contact — hide and clear, do not submit
            $('#owner_info_form').hide();
            $('#first_name_1, #last_name_1, #email_1').prop('required', false).val('');
        } else {
            // Owner is a different person — show and require
            $('#owner_info_form').show();
            $('#first_name_1, #last_name_1, #email_1').prop('required', true);
        }
    });

    // Initialize visibility (default: primary_contact=1)
    $('#owner_info_form').hide();
});
```

### Form Submission - Omit Fields When primary_contact=1

```javascript
$('#ownershipForm').on('submit', function(e) {
    e.preventDefault();

    const primaryContact = $('input[name="primary_contact"]:checked').val();
    const formData = new FormData(this);

    // If primary_contact=1, remove first_name[1]/last_name[1]/email[1] entirely —
    // the backend uses the Step 1 user's data automatically
    if (primaryContact === '1') {
        formData.delete('first_name[1]');
        formData.delete('last_name[1]');
        formData.delete('email[1]');
    }
    // If primary_contact=0, the visible/required inputs above are submitted as-is

    // Submit form
    $.ajax({
        url: '/api/v1/ownership',
        type: 'POST',
        headers: { 'X-API-Key': apiKey },
        data: formData,
        processData: false,
        contentType: false,
        success: function(response) {
            if (response.status) {
                goToStep(5, uuid); // however the implementation navigates
            }
        }
    });
});
```

---

**Continue to**: [STEP4_OWNER_INFORMATION_FIELDS.md](STEP4_OWNER_INFORMATION_FIELDS.md) for the full Form Submission Format examples, Owner 1/2 field tables, validation rules, and API responses.
