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
| first_name | text | Yes | Max 60 chars |
| last_name | text | Yes | Max 60 chars |
| email | email | Yes | Valid email format |
| phone | tel | Yes | Use intl-tel-input library |
| name | text | Yes | Company name, max 60 chars |
| website | url | Yes | Valid URL or social media profile |
| country | select | Yes | **SEARCHABLE** - API: `/api/partner/countries` with live search |
| business_state | select | Yes | **SEARCHABLE** - Show only if country="US", API: `/api/partner/states` with live search |
| annual_sales | number | Yes | Numeric only, no currency symbols or commas |
| promo_code | text | No | Optional referral code |

---

## Country Dropdown

**Field**: `country`  
**Type**: SELECT dropdown  
**API Endpoint**: `GET /api/partner/countries`  
**Header**: `Authorization: {user.security_key}`

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

---

## API Integration

**Dropdown Data**:
```
GET /api/partner/countries
GET /api/partner/states
```

Header: `Authorization: {user.security_key}`

**Form Submission**:
```
POST /api/v1/signup
Headers: X-API-Key: {user.security_key}
Payload: 
  country: "US" (use code/slug, not id)
  all other Step 1 fields
  step_count: 1
Response: { uuid, step_count, message }
Store UUID in localStorage for all subsequent steps
```

---

## Validation

- First/Last name: required, max 60 chars
- Email: required, valid format
- Phone: required, valid number (intl-tel-input validates)
- Website: required, valid URL
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

// Store UUID for all subsequent steps
localStorage.setItem('signup_uuid', response.uuid);

// Redirect to Step 2
window.location.href = `/signup/step/2/${response.uuid}`;
```

**On Validation Error (HTTP 422)**:
```javascript
// Response example:
{
  "status": false,
  "message": "Validation failed",
  "errors": {
    "first_name": ["First name is required"],
    "email": ["Email must be a valid email"]
  }
}

// Display field errors
// DO NOT change URL - user stays on Step 1
for (const [field, messages] of Object.entries(response.errors)) {
  displayErrorForField(field, messages[0]);
}
```

---

## Field Summary

**Total Fields**: 10  
**Required Fields**: 9 (all except promo_code)  
**Conditionally Required**: 1 (business_state - required if country=US)  
**Phone Fields**: 1 (intl-tel-input)

---

**Production Ready** ✅
