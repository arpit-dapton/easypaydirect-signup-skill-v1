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

## API Endpoint

**POST `/v1/ownership`**

**Authentication**: X-API-Key header (from Step 1)

**Content-Type**: application/x-www-form-urlencoded

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
- **HIDE** `first_name[1]`, `last_name[1]`, `email[1]` fields 
- **DO NOT SUBMIT** these fields to API (use user's Step 1 data instead)
- Backend uses Step 1 first_name, last_name, email from User record
- Set `primary_contact_user_id` = newly created Owner 1 user ID

**If primary_contact=0 (NOT owner)**:
- Show `primary_contact_job_title` field (REQUIRED)
- **SHOW** `first_name[1]`, `last_name[1]`, `email[1]` fields (user editable)
- User manually enters Owner 1 info
- Keep `primary_contact_user_id` unchanged from Step 1

---

## Form Submission Format

**POST /v1/ownership** with array notation:

⚠️ **CRITICAL: ALL fields MUST use consistent `[1]` bracket notation** (NOT object `{1: value}` format)
- ✅ CORRECT: `bankruptcy_filed[1]=1`, `bankruptcy_discharged[1]=0`, `bankruptcy_discharged_date[1]=2008-05-23`
- ❌ WRONG: `bankruptcy_discharged_date={1: "2008-05-23", 2: ""}`

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
| ssn | text | ssn[1] | required | Masked, encrypted in DB |
| dob | date | dob[1] | required | YYYY-MM-DD format |
| ownership_percentage | number | ownership_percentage[1] | required, 1-100 | Triggers Owner 2 visibility if < 51% |

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

⚠️ **All owner fields must use bracket array notation `[1]` or `[2]`, NOT object notation `{1: value}`**

```javascript
$('#ownership-form').on('submit', function(e) {
    e.preventDefault();
    
    const uuid = localStorage.getItem('signup_uuid');
    const apiKey = sessionStorage.getItem('api_key');
    const formData = new FormData(this);
    const primaryContact = $('input[name="primary_contact"]:checked').val();
    
    // CRITICAL: Ensure all fields use bracket notation [1] or [2]
    // NOT object notation like {1: value, 2: value}
    
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
        url: '/v1/ownership',
        type: 'POST',
        headers: { 'X-API-Key': apiKey },
        data: formData,
        processData: false,
        contentType: false,
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
  SHOW: first_name[1], last_name[1], email[1] (EDITABLE)
  SUBMIT: first_name[1], last_name[1], email[1] to API
  
ELSE (primary_contact = 1, IS owner):
  HIDE: primary_contact_job_title
  HIDE: first_name[1], last_name[1], email[1] (COMPLETELY HIDDEN)
  DO NOT SUBMIT: first_name[1], last_name[1], email[1] to API
  Backend uses Step 1 user data for these fields
```

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
