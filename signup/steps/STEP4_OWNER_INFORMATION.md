# Step 4: Owner Information

Complete specification for collecting business owner information including primary contact status, owner details, addresses, and financial history.

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
- Google Maps address autocomplete
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
- **HIDE** `first_name[1]`, `last_name[1]`, `email[1]` fields from user input
- **PRE-FILL (READ-ONLY)** these fields with Step 1 user data:
  - `first_name[1]` = User's first_name from Step 1
  - `last_name[1]` = User's last_name from Step 1
  - `email[1]` = User's email from Step 1
- **DO NOT SUBMIT** these fields to API (backend uses Step 1 data instead)
- Backend uses Step 1 first_name, last_name, email from User record
- Set `primary_contact_user_id` = newly created Owner 1 user ID

**If primary_contact=0 (NOT owner)**:
- Show `primary_contact_job_title` field (REQUIRED)
- **SHOW** `first_name[1]`, `last_name[1]`, `email[1]` fields (user editable)
- User manually enters Owner 1 info
- Keep `primary_contact_user_id` unchanged from Step 1

---

## Pre-fill Step 1 Data When primary_contact=1

⚠️ **When primary_contact=1, MUST pre-fill first_name[1], last_name[1], email[1] from Step 1 user data**

### How to Retrieve Step 1 User Data

**Option 1: From Stored User Session/LocalStorage**:
```javascript
// Store user data in Step 1 after form submission
localStorage.setItem('step1_user_data', JSON.stringify({
    first_name: response.first_name,
    last_name: response.last_name,
    email: response.email,
    phone: response.phone
}));
```

**Option 2: From API Call to User Endpoint** (if Step 1 data not stored):
```javascript
// Fetch user data from backend
$.ajax({
    url: `/api/partner/user/${uuid}`,
    type: 'GET',
    headers: { 'Authorization': apiKey },
    success: function(response) {
        const userData = response.data;
        preFillOwnerData(userData);
    }
});
```

### HTML - Owner 1 Basic Info (with hidden inputs)

```html
<!-- Hidden inputs that store Step 1 data - NOT visible to user -->
<input type="hidden" name="first_name[1]" id="first_name_1" value="">
<input type="hidden" name="last_name[1]" id="last_name_1" value="">
<input type="hidden" name="email[1]" id="email_1" value="">

<!-- Display-only divs showing the pre-filled data -->
<div id="owner_info_display" style="display: none;">
    <div class="alert alert-info">
        <p><strong>Owner Information:</strong></p>
        <p><strong>Name:</strong> <span id="display_name"></span></p>
        <p><strong>Email:</strong> <span id="display_email"></span></p>
        <small>This information is taken from your Step 1 signup data</small>
    </div>
</div>

<!-- Editable form fields - show when primary_contact=0 -->
<div id="owner_info_form" style="display: none;">
    <div class="form-group">
        <label for="first_name_input_1">First Name *</label>
        <input type="text" id="first_name_input_1" name="first_name_edit[1]" 
               class="form-control" required>
    </div>
    <div class="form-group">
        <label for="last_name_input_1">Last Name *</label>
        <input type="text" id="last_name_input_1" name="last_name_edit[1]" 
               class="form-control" required>
    </div>
    <div class="form-group">
        <label for="email_input_1">Email *</label>
        <input type="email" id="email_input_1" name="email_edit[1]" 
               class="form-control" required>
    </div>
</div>
```

### JavaScript - Pre-fill and Toggle Display

```javascript
$(document).ready(function() {
    const apiKey = sessionStorage.getItem('api_key');
    const uuid = sessionStorage.getItem('signup_uuid');
    
    // Fetch Step 1 user data
    function fetchStep1UserData() {
        $.ajax({
            url: `/api/v1/user/${uuid}`,
            type: 'POST',
            headers: { 'X-API-Key': apiKey },
            success: function(response) {
                if (response.status && response.data) {
                    const userData = response.data;
                    preFillOwnerData(userData);
                }
            },
            error: function() {
                // Fallback: try localStorage
                const stored = localStorage.getItem('step1_user_data');
                if (stored) {
                    const userData = JSON.parse(stored);
                    preFillOwnerData(userData);
                }
            }
        });
    }
    
    // Pre-fill hidden inputs with Step 1 data
    function preFillOwnerData(userData) {
        $('#first_name_1').val(userData.first_name || '');
        $('#last_name_1').val(userData.last_name || '');
        $('#email_1').val(userData.email || '');
        
        // Display the pre-filled data
        $('#display_name').text(userData.first_name + ' ' + userData.last_name);
        $('#display_email').text(userData.email);
    }
    
    // Toggle display based on primary_contact selection
    $('input[name="primary_contact"]').on('change', function() {
        const isPrimaryContact = $(this).val() === '1';
        
        if (isPrimaryContact) {
            // SHOW: Display-only info (pre-filled from Step 1)
            $('#owner_info_display').show();
            $('#owner_info_form').hide();
            
            // Clear and disable editable fields
            $('#first_name_input_1, #last_name_input_1, #email_input_1').val('');
        } else {
            // SHOW: Editable form fields
            $('#owner_info_display').hide();
            $('#owner_info_form').show();
            
            // Enable editing
            $('#first_name_input_1, #last_name_input_1, #email_input_1').prop('required', true);
        }
    });
    
    // On form load, fetch and pre-fill Step 1 data
    fetchStep1UserData();
    
    // Initialize visibility (default: primary_contact=1)
    $('#owner_info_display').show();
    $('#owner_info_form').hide();
});
```

### Form Submission - Handle Both Cases

```javascript
$('#ownershipForm').on('submit', function(e) {
    e.preventDefault();
    
    const primaryContact = $('input[name="primary_contact"]:checked').val();
    const formData = new FormData(this);
    
    // If primary_contact=1, use hidden inputs (pre-filled from Step 1)
    // If primary_contact=0, use editable form inputs
    if (primaryContact === '0') {
        // User is not owner - submit edited data
        formData.set('first_name[1]', $('#first_name_input_1').val());
        formData.set('last_name[1]', $('#last_name_input_1').val());
        formData.set('email[1]', $('#email_input_1').val());
    }
    // If primaryContact === '1', hidden inputs already have correct values
    
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
                window.location.href = `/signup/step/5/${uuid}`;
            }
        }
    });
});
```

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
       headers: { 'X-API-Key': apiKey },
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

```
uuid=550e8400-e29b-41d4-a716-446655440000
primary_contact=1
primary_contact_job_title=
first_name[1]=                    (EMPTY - use Step 1 data)
last_name[1]=                     (EMPTY - use Step 1 data)
email[1]=                         (EMPTY - use Step 1 data)
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
| first_name | text | first_name[1] | required, max:60 | **HIDDEN if primary_contact=1**, visible/editable if primary_contact=0 |
| last_name | text | last_name[1] | required, max:60 | **HIDDEN if primary_contact=1**, visible/editable if primary_contact=0 |
| email | email | email[1] | required, valid email | **HIDDEN if primary_contact=1**, visible/editable if primary_contact=0 |
| phone | tel | phone[1] | required | intl-tel-input format |
| title | select | title[1] | required | **Use slug values**: CEO, CFO, Chairman, Co-owner, Controller, Director, General-Manager, Office-Manager, Owner, Partner, President, Treasurer, Vice-President, Other |
| ssn | text | ssn[1] | required | **Format: XXX-XX-XXXX** (9 digits with dashes), NOT masked in input |
| dob | date | dob[1] | required | YYYY-MM-DD format |
| ownership_percentage | number | ownership_percentage[1] | required, 1-100 | Triggers Owner 2 visibility if < 51% |

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

### Address (with Google Maps Autocomplete)

| Field | Type | Name | ID | Validation | Notes |
|-------|------|------|-----|------------|-------|
| address_search | text | N/A | autocomplete2 | optional | Google Places autocomplete |
| street_number | text | street_number[1] | owner_1_street_number | required | Auto-filled from autocomplete |
| street_address | text | street_address[1] | owner_1_street_address | required | Auto-filled from autocomplete |
| city | text | city[1] | owner_1_city | required | Auto-filled from autocomplete |
| state | text | state[1] | owner_1_state | required | State code, auto-filled |
| postal_code | text | postal_code[1] | owner_1_postal_code | required | Auto-filled from autocomplete |
| country | select | country[1] | owner_country1 | required | selectpicker, auto-filled |

### License (Conditional - Required if Country=US)

| Field | Type | Name | Validation | Condition |
|-------|------|------|------------|-----------|
| license | text | license[1] | required, 5-25 chars | Driver's License/Passport |
| driver_license_state | select | driver_license_state[1] | required if country=US | US states only |
| driver_license_expiration_date | date | driver_license_expiration_date[1] | required if country=US | Future date |

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
- Google autocomplete: `id="autocomplete3"`
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
'dob.1' => 'required|date_format:Y-m-d',
'ownership_percentage.1' => 'required|integer|min:1|max:100',

// Address
'street_number.1' => 'required|string|max:10',
'street_address.1' => 'required|string|max:255',
'city.1' => 'required|string|max:100',
'state.1' => 'required|string|max:2|exists:us_states,code',
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
- If `country=US`: Driver license fields required
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

**Action**: Redirect to Step 5: `/signup/step/5/{uuid}`

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

**Headers**: X-API-Key, Content-Type: application/json

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

## Libraries Required

- Date picker library (e.g., Flatpickr, Bootstrap DatePicker, or similar) - For dob, driver_license_expiration_date, bankruptcy_discharged_date
- Google Maps Places API - Address autocomplete
- `intl-tel-input` - Phone formatting

---

## Google Maps Autocomplete

**Script**:
```html
<script src="https://maps.google.com/maps/api/js?key={KEY}&libraries=places"></script>
```

**Input Fields**: `id="autocomplete2"` (Owner 1), `id="autocomplete3"` (Owner 2)

**Output Fields**:

Owner 1:
- `owner_1_street_number`
- `owner_1_street_address`
- `owner_1_city`
- `owner_1_state`
- `owner_1_postal_code`
- `owner_country1`

Owner 2:
- `owner_2_street_number`
- `owner_2_street_address`
- `owner_2_city`
- `owner_2_state`
- `owner_2_postal_code`
- `owner_country2`

**Behavior**:
1. User selects place from autocomplete
2. All 6 address fields auto-populate
3. Autocomplete input hides
4. Address fields become visible and required

---

## JavaScript

### Form Submission

⚠️ **CRITICAL: MUST send as FormData (application/x-www-form-urlencoded), NOT JSON**

All owner fields must use bracket array notation `[1]` or `[2]`, NOT object notation `{1: value}`

**Correct Implementation**:
```javascript
$('#ownership-form').on('submit', function(e) {
    e.preventDefault();
    
    const uuid = localStorage.getItem('signup_uuid');
    const apiKey = sessionStorage.getItem('api_key');
    
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
        headers: { 'X-API-Key': apiKey },
        data: formData,
        processData: false,        // ← Critical: tells jQuery to NOT process FormData
        contentType: false,        // ← Critical: lets browser set multipart/form-data
        // ❌ NEVER: contentType: 'application/json'
        // ❌ NEVER: JSON.stringify(formData)
        
        success: function(response) {
            if (response.status) {
                window.location.href = `/signup/step/5/${uuid}`;
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
    headers: { 'X-API-Key': apiKey },
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
X-API-Key: user-api-key

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
  HIDE: Editable form fields for first_name[1], last_name[1], email[1]
  SHOW: Display-only info box with PRE-FILLED Step 1 data
  SUBMIT: hidden inputs with Step 1 data (first_name[1], last_name[1], email[1])
  Backend uses these Step 1 values for Owner 1
```

**Key Points**:
- When primary_contact=1: Show "Owner information is taken from Step 1 data" message
- When primary_contact=0: Show editable form fields for owner data
- Hidden inputs always exist to store the correct data
- Form submission uses appropriate data based on primary_contact value

### Owner 2 Section
```
IF ownership_percentage[1] < 51 → SHOW
ELSE → HIDE & CLEAR
```

### Driver License Fields
```
IF country[1] = "US" → SHOW (REQUIRED)
ELSE → HIDE & CLEAR
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
- [ ] Google autocomplete populates address
- [ ] country=US: Shows driver license fields
- [ ] country≠US: Hides driver license fields
- [ ] bankruptcy_filed=1: Shows discharged field
- [ ] bankruptcy_discharged=1: Shows date field
- [ ] All required fields validated
- [ ] Redirects to Step 5 on success
- [ ] Shows errors on validation failure

---

**Status**: Production Ready
