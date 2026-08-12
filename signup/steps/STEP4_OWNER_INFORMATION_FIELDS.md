---
name: signup-step-4-fields
description: Step 4 Owner Information - Submission payload shape, Owner 1/2 field tables, validation rules, and API responses
---

# Step 4: Owner Information — Field Reference

Continuation of [STEP4_OWNER_INFORMATION.md](STEP4_OWNER_INFORMATION.md) (naming format, FormData requirement, date pickers, primary contact). This file covers the full submission payload shape, Owner 1/2 field tables, validation rules, and API responses.

**Continue to**: [STEP4_OWNER_INFORMATION_IMPLEMENTATION.md](STEP4_OWNER_INFORMATION_IMPLEMENTATION.md) for libraries, JavaScript, conditional logic, and the testing checklist.

---

## Form Submission Format

**POST /api/v1/ownership** - MUST use FormData (application/x-www-form-urlencoded)

⚠️ **CRITICAL INSTRUCTIONS** (Follow ALL 4):

1. **Use FormData - NOT JSON**:
   ```javascript
   // ✅ CORRECT
   const formData = new FormData();
   formData.append('first_name[1]', 'John');
   formData.append('last_name[1]', 'Doe');

   // ❌ WRONG - Do NOT do this:
   JSON.stringify({...})
   contentType: 'application/json'
   ```

2. **ALL fields MUST use bracket notation** `[1]` or `[2]` (NOT object `{1: value}`):
   ```
   ✅ CORRECT FORMAT:
   bankruptcy_filed[1]=1
   bankruptcy_discharged[1]=0
   bankruptcy_discharged_date[1]=2008-05-23
   first_name[1]=John

   ❌ WRONG FORMAT:
   bankruptcy_discharged_date={1: "2008-05-23", 2: ""}
   {first_name[1]: "John"}
   ```

3. **AJAX MUST disable content-type processing**:
   ```javascript
   $.ajax({
       url: '/api/v1/ownership',
       type: 'POST',
       data: formData,
       processData: false,    // ← REQUIRED
       contentType: false,    // ← REQUIRED (lets browser send as form-data)
       // ❌ NEVER use: contentType: 'application/json'
   });
   ```

4. **How Laravel receives it**:
   ```
   Form Data: first_name[1]=John
                ↓
   Laravel parses: first_name => [1 => "John"]
                ↓
   Validation accesses: first_name.1 ✅
   ```

### If primary_contact=1 (User IS the owner):

⚠️ **Do NOT include `first_name[1]`, `last_name[1]`, `email[1]` in the payload at all.** Omit them entirely — do not send them as empty strings. The backend automatically fills Owner 1's name and email from the Step 1 user record when `primary_contact=1`.

```
uuid=550e8400-e29b-41d4-a716-446655440000
primary_contact=1
primary_contact_job_title=
                                   (first_name[1], last_name[1], email[1] OMITTED — not sent)
phone[1]=+1-555-1234
title[1]=CEO
ssn[1]=123-45-6789
dob[1]=1980-01-15
ownership_percentage[1]=100
street_number[1]=123
street_address[1]=Main Street
city[1]=New York
state[1]=NY
postal_code[1]=10001
country[1]=US
license[1]=DL123456789
driver_license_state[1]=NY
driver_license_expiration_date[1]=2030-12-31
bankruptcy_filed[1]=0
bankruptcy_discharged[1]=0
bankruptcy_discharged_date[1]=
step_count=4
```

### If primary_contact=0 (User is NOT the owner):

```
uuid=550e8400-e29b-41d4-a716-446655440000
primary_contact=0
primary_contact_job_title=CEO     (REQUIRED when primary_contact=0)
first_name[1]=John                (REQUIRED - user enters owner info)
last_name[1]=Doe                  (REQUIRED - user enters owner info)
email[1]=john@example.com         (REQUIRED - user enters owner info)
phone[1]=+1-555-1234
title[1]=CFO
ssn[1]=123-45-6789
dob[1]=1980-01-15
ownership_percentage[1]=60
street_number[1]=123
street_address[1]=Main Street
city[1]=New York
state[1]=NY
postal_code[1]=10001
country[1]=US
license[1]=DL123456789
driver_license_state[1]=NY
driver_license_expiration_date[1]=2030-12-31
bankruptcy_filed[1]=0
bankruptcy_discharged[1]=0
bankruptcy_discharged_date[1]=
step_count=4
```

---

## Owner 1 Fields

### Basic Information (All Required)

| Field | Type | Name | Validation | Notes |
|-------|------|------|------------|-------|
| first_name | text | first_name[1] | required if primary_contact=0 | **OMIT entirely if primary_contact=1** (not hidden-with-value — simply not rendered/submitted), visible/editable/required if primary_contact=0 |
| last_name | text | last_name[1] | required if primary_contact=0 | **OMIT entirely if primary_contact=1**, visible/editable/required if primary_contact=0 |
| email | email | email[1] | required if primary_contact=0 | **OMIT entirely if primary_contact=1**, visible/editable/required if primary_contact=0. **🔒 Disabled (along with all other Step 4 fields) after Step 4 is successfully submitted** — see STEP4_OWNER_INFORMATION_IMPLEMENTATION.md → "Full Field Lock After Submission" |
| phone | tel | phone[1] | required | intl-tel-input format |
| title | select | title[1] | required | **Use slug values**: CEO, CFO, Chairman, Co-owner, Controller, Director, General-Manager, Office-Manager, Owner, Partner, President, Treasurer, Vice-President, Other |
| ssn | text | ssn[1] | required | **Format: XXX-XX-XXXX** (9 digits with dashes), NOT masked in input |
| dob | date | dob[1] | required | YYYY-MM-DD format. **Owner must be at least 18 years old.** Enforce client-side by setting Flatpickr `maxDate` to 18 years before today (see [STEP4_OWNER_INFORMATION.md](STEP4_OWNER_INFORMATION.md) → "Date Picker Fields"). Backend validation: `required\|date_format:Y-m-d\|before_or_equal:<18-years-ago>`. |
| ownership_percentage | number | ownership_percentage[1] | required, 1-100 | Triggers Owner 2 visibility if < 51%. If Owner 2 is present, `ownership_percentage[1] + ownership_percentage[2]` must not exceed 100. **🔒 Disabled (along with all other Step 4 fields) after Step 4 is successfully submitted** — see STEP4_OWNER_INFORMATION_IMPLEMENTATION.md → "Full Field Lock After Submission" |

### SSN Input - Social Security Number

⚠️ **IMPORTANT: SSN should NOT be masked. Format input as XXX-XX-XXXX**

**Field**: `ssn[1]`, `ssn[2]`
**Type**: TEXT input with auto-formatting
**Format**: XXX-XX-XXXX (e.g., 123-45-6789)
**Required**: Yes
**Validation**: 9 numeric digits (dashes added automatically)

**HTML**:
```html
<div class="form-group">
    <label for="ssn_1">Social Security Number (Owner 1)</label>
    <input type="text" id="ssn_1" name="ssn[1]" class="form-control"
           placeholder="XXX-XX-XXXX" maxlength="11" required
           inputmode="numeric" pattern="[0-9\-]{11}">
</div>
```

**JavaScript - Auto-format on input**:
```javascript
// Format SSN as user types: XXX-XX-XXXX
$('#ssn_1').on('input', function() {
    let value = $(this).val().replace(/\D/g, ''); // Remove non-digits

    if (value.length > 0) {
        if (value.length <= 3) {
            value = value;
        } else if (value.length <= 5) {
            value = value.substring(0, 3) + '-' + value.substring(3);
        } else {
            value = value.substring(0, 3) + '-' + value.substring(3, 5) + '-' + value.substring(5, 9);
        }
    }

    $(this).val(value);
});
```

**Validation**:
- Must contain exactly 9 numeric digits
- Format automatically as user types
- Backend validation: `'ssn.1' => 'required|string'`
- Digits extracted before storage: 123456789 (stored without dashes in database)

**Security**:
- SSN is encrypted in the database (no display/masking in UI)
- Input field does NOT mask/hide the typed characters
- Users can see what they're typing for verification purposes

---

### Address (Manual Entry)

| Field | Type | Name | ID | Validation | Notes |
|-------|------|------|-----|------------|-------|
| street_number | text | street_number[1] | owner_1_street_number | required | |
| street_address | text | street_address[1] | owner_1_street_address | required | |
| city | text | city[1] | owner_1_city | required | |
| state | text | state[1] | owner_1_state | required | **Free text** — NOT restricted to US states, no country-based validation. See ⚠️ note below. |
| postal_code | text | postal_code[1] | owner_1_postal_code | required | |
| country | select | country[1] | owner_country1 | required | normal dropdown (no search), API: `/api/partner/countries` |

⚠️ **`state[1]`/`state[2]` must be a plain free-text input, valid for any country — do NOT feed it from a US-states-only endpoint/dropdown.** This is easy to get wrong because it's easily confused with the separate `driver_license_state[i]` field below, which genuinely *is* US-states-only, but only because it's conditional on `country[i]="US"` in the first place. The general owner `state[i]` address field has no such restriction at any layer — it accepts any string for any owner country.

### License

⚠️ **`license` is always visible and required for every country.** Only `driver_license_state` and `driver_license_expiration_date` are conditional — visible/required for `country=US`, hidden for every other country.

| Field | Type | Name | Validation | Condition |
|-------|------|------|------------|-----------|
| license | text | license[1] | required, 5-25 chars | **Always visible, all countries** — Driver's License/Passport |
| driver_license_state | select | driver_license_state[1] | required if country=US | **Visible only if country=US**, hidden otherwise — US states only |
| driver_license_expiration_date | date | driver_license_expiration_date[1] | required if country=US | **Visible only if country=US**, hidden otherwise — future date |

### Bankruptcy (Cascade Logic)

| Field | Type | Name | Validation | Condition |
|-------|------|------|------------|-----------|
| bankruptcy_filed | radio | bankruptcy_filed[1] | required | 0 or 1 |
| bankruptcy_discharged | radio | bankruptcy_discharged[1] | required if bankruptcy_filed=1 | 0 or 1 |
| bankruptcy_discharged_date | date | bankruptcy_discharged_date[1] | required if bankruptcy_discharged=1 | Past date |

---

## Owner 2 Fields (Conditional)

**Visibility**: Show ONLY if `ownership_percentage[1] < 51%`

Same structure as Owner 1, with `[2]` suffix:
- `first_name[2]`, `last_name[2]`, `email[2]`, etc.
- Country select: `id="owner_country2"`

**Note**: All Owner 2 fields optional. Collected only if data is provided.

---

## Validation Rules

### Owner 1 (Required)

```php
'primary_contact' => 'required|in:0,1',
'primary_contact_job_title' => 'required_if:primary_contact,0',

// Basic info
'first_name.1' => 'required|string|max:60',
'last_name.1' => 'required|string|max:60',
'email.1' => 'required|email',
'phone.1' => 'required|string',
'title.1' => 'required|string|exists:owner_job_title,slug',
'ssn.1' => 'required|string',
'dob.1' => 'required|date_format:Y-m-d|before_or_equal:' . now()->subYears(18)->format('Y-m-d'), // minimum age 18
'ownership_percentage.1' => 'required|integer|min:1|max:100',

// Address
'street_number.1' => 'required|string|max:10',
'street_address.1' => 'required|string|max:255',
'city.1' => 'required|string|max:100',
'state.1' => 'required|string', // free text — no us_states/country constraint, any owner country
'postal_code.1' => 'required|string|max:20',
'country.1' => 'required|string|exists:country,country_code',

// License
'license.1' => 'required|string|min:5|max:25',
'driver_license_state.1' => 'nullable|string|max:2|exists:us_states,code',
'driver_license_expiration_date.1' => 'nullable|date_format:Y-m-d|after_or_equal:today',

// Bankruptcy
'bankruptcy_filed.1' => 'required|in:0,1',
'bankruptcy_discharged.1' => 'nullable|in:0,1',
'bankruptcy_discharged_date.1' => 'nullable|date_format:Y-m-d|before_or_equal:today',
```

### Owner 2 (Optional - if ownership < 51%)

Same as Owner 1 with `[2]` suffix, all fields nullable.

### Conditional Rules

- If `primary_contact=0`: `primary_contact_job_title` required
- If `ownership_percentage.1 < 51`: Owner 2 fields required (if first_name[2] filled)
- **If Owner 2 is present**: `ownership_percentage[1] + ownership_percentage[2]` must not exceed 100. Validate this client-side as the user types (e.g. cap Owner 2's max input at `100 - ownership_percentage[1]`) as well as on submit.
- `license` is required for all countries; if `country=US`: `driver_license_state` and `driver_license_expiration_date` are also required (hidden for every other country)
- If `bankruptcy_discharged=1`: `bankruptcy_discharged_date` required

---

## API Responses

### Success (HTTP 200)

```json
{
  "status": true,
  "message": "Owners created successfully",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "ruleEngineResponse": { ... }
}
```

**Action**: Proceed to Step 5, passing the same `uuid`.

### Validation Error (HTTP 422)

```json
{
  "status": false,
  "message": "Validation failed",
  "errors": {
    "first_name.1": ["First name is required"],
    "country.1": ["Country must be valid"],
    "primary_contact_job_title": ["Job title is required"]
  }
}
```

**Action**: Stay on Step 4, display errors

---

## User Info API (Optional)

**NOT automatically called by backend**. This is an optional endpoint IF frontend needs to fetch current user data.

**POST `/v1/user/{dealUuid}`**

**Headers**: Content-Type: application/json

**Response**:
```json
{
  "status": true,
  "data": {
    "uuid": "user-uuid",
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com",
    "phone": "+1-555-1234"
  }
}
```

**Note**: Backend does NOT call this automatically. Frontend must decide whether to:
- Pre-populate hidden fields when primary_contact=1
- OR leave fields empty (user data still submitted as-is)
- Backend accepts whatever values are sent in first_name[1], last_name[1], email[1]

---

**Continue to**: [STEP4_OWNER_INFORMATION_IMPLEMENTATION.md](STEP4_OWNER_INFORMATION_IMPLEMENTATION.md) for libraries, JavaScript, conditional logic, and the testing checklist.

**See also**: [STEP4_OWNER_INFORMATION.md](STEP4_OWNER_INFORMATION.md) for naming format, FormData requirement, date pickers, and primary contact handling.
