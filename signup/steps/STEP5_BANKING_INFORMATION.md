---
name: signup-step-5
description: Step 5 Banking Information - Bank account details and payment routing information
---

# STEP 5: Banking Information

Fifth step of the 6-step signup form. Collects bank account and routing information for payment processing.

---

## Fields

> ℹ️ **`country` is a Step 1 field** (see [STEP1_ACCOUNT_INFORMATION.md](STEP1_ACCOUNT_INFORMATION.md)). Step 5 does **NOT** render a country dropdown. The Canada-only conditionals below read the persisted Step 1 `country` value (`$company->country`, using the code/slug — `"US"`, `"CA"` — not the numeric `id`). Do **not** add a `#country` `<select>` or a `$('#country').on('change')` handler on this step — evaluate the persisted country **once on page load**.

> ℹ️ **No account-type field.** Bank account type is **not** collected in the UI. The client sends `bank_account_type` programmatically (defaults to `1`); do **not** render a Checking/Savings dropdown or textbox.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| institution_number | text | Conditional | Show if Step 1 `country="CA"` (Canada), 3 digits |
| customer_pay_currency | radio | Conditional | Show if Step 1 `country="CA"` (Canada), Options: USD, CAD |
| routing_number | text | Yes | Label varies by Step 1 `country` |
| account_number | text | Yes | Alphanumeric |
| current_processing | select | No | Options: Yes(1), No(0) |
| processor_name | text | Conditional | Show if current_processing=1, max 300 chars |

---

## Field Labels by Country

**Routing Number**:
- US: "Routing Number" (9 digits)
- Canada: "Transit Number" (5 digits)
- UK: "Sort Code" (6 digits)
- Australia: "BSB" (6 digits)

**Account Number**:
- US: "Account Number" (max 17 chars)
- Canada: "Account Number" (max 12 chars)
- Others: "Account Number" (max 34 chars)

---

## Dependent Fields

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`).

- **institution_number**: Show if Step 1 `country="CA"` (Canada) — read persisted value, no `#country` dropdown on this step
- **customer_pay_currency**: Show if Step 1 `country="CA"` (Canada)
- **processor_name**: Show if current_processing=1

**Country-based visibility** — evaluate the persisted Step 1 country once on load (server-side `$company->country`, or inject it into the page). Example:
```javascript
// country comes from Step 1 (persisted), NOT a Step 5 <select>
var companyCountry = "{{ $company->country }}"; // code/slug, e.g. "US" or "CA"
if (companyCountry === "CA") { // Canada
    $('#institution_number_wrapper').show();
    $('#institution_number').prop('required', true);
    $('.canadian_currency_div').show();
    $('input[name="customer_pay_currency"]').prop('required', true);
}
```

---

## Form Submission & Redirect

**On Success (HTTP 200/201)**:
```javascript
// Response example:
{
  "status": true,
  "message": "Banking information saved successfully",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "step_count": 5
}

// UUID already obtained from Step 1 — pass it forward to Step 6
// (storage mechanism is the implementer's choice — see skill.md Guidance section)
```

**On Validation Error (HTTP 422)**:
```javascript
// Response example:
{
  "status": false,
  "message": "Validation failed",
  "errors": {
    "routing_number": ["Routing number is invalid"],
    "account_number": ["Account number is required"]
  }
}

// Display field errors
// DO NOT change URL - user stays on Step 5
for (const [field, messages] of Object.entries(response.errors)) {
  displayErrorForField(field, messages[0]);
}
```

---

## API Integration

**Form Submission**:
```
POST /api/v1/application/step
Headers: None required (see skill.md → Authentication)
Payload:
  step_count: 5
  section: banking_info
  all Step 5 fields
  uuid: {uuid}
Response: { next_step_url, message }
```

---

## Field Summary

**Total Fields**: 9 (country is a Step 1 field; no account-type UI field)  
**Required Fields**: 5  
**Conditional Fields**: 3

---

**Production Ready** ✅
