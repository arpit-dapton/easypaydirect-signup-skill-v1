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

// Persist response.uuid for all subsequent steps (storage mechanism is your choice —
// see skill.md → Guidance) and proceed to Step 2
```

**On Validation Error (HTTP 422)**: standard shape — see skill.md → Error Handling. Stay on Step 1, display field errors, do not change URL.

---

## Field Summary

**Total Fields**: 11  
**Required Fields**: 9 (all except promo_code and partner_key)  
**Conditionally Required**: 1 (business_state - required if country=US)  
**Optional**: 2 (promo_code, partner_key)  
**Phone Fields**: 1 (intl-tel-input)
