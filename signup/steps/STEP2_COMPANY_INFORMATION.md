---
name: signup-step-2
description: Step 2 Company Information - Business details, legal address, and revenue model
---

# STEP 2: Company Information

Second step of the 6-step signup form. Collects company business details, addresses, and revenue model information.

**Continue to**: [STEP2_COMPANY_INFORMATION_CONDITIONALS.md](STEP2_COMPANY_INFORMATION_CONDITIONALS.md) for dependent field logic, Federal Tax ID formatting, form submission, and API integration.

---

## Fields

**Company Details** (9 fields):

> ℹ️ **`country` and `business_state` are Step 1 fields** (see [STEP1_ACCOUNT_INFORMATION.md](STEP1_ACCOUNT_INFORMATION.md)). They are **NOT** collected in Step 2. The Step 2 fields below that vary by country (`federal_tax_id`, `business_register_number`) read the `country` value the merchant selected in Step 1, which persists on the application/company record.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| legal_name | text | Yes | Max 60 chars. **Pre-fill from Step 1 `name` field** |
| name | text | Yes | DBA name, max 60 chars. **Pre-fill from Step 1 `name` field** |
| industry_type | select | Yes | Normal dropdown (no search) - API: `/api/partner/industry-types` |
| industry_type_other | text | Conditional | Show if industry_type="other", max 255 chars |
| customer_service_telephone_number | tel | Yes | Use intl-tel-input |
| business_location | select | Yes | Static (slug): Home-Based, Co-Working, Corporate-Office, Storefront, Others |
| business_formed | date | Yes | YYYY-MM-DD, use date picker |
| business_organized | select | Yes | Static (slug): Corporation, LLC, Partnership, Government, Sole-Proprietorship, Non-Profit, Other |
| federal_tax_id | text | Conditional | Required ONLY if Step 1 `country`="US", max 20 chars, encrypted. Formatted input — see **Federal Tax ID Input Format** in [STEP2_COMPANY_INFORMATION_CONDITIONALS.md](STEP2_COMPANY_INFORMATION_CONDITIONALS.md). Label/format vary by Step 1 `country` |
| business_register_number | text | Conditional | Required ONLY if Step 1 `country`≠"US" (not US), max 20 chars (11 for Canada) |

**Revenue Model** (3 fields - Checkbox Array with Dependent Hierarchy):

| Field | Type | Required | Name Attribute | Notes |
|-------|------|----------|---|---|
| marketingModel | checkbox array | Yes | marketingModel[] | Options: "One Time Purchase" (1), "Recurring/Subscription" (2), "Trial Offer + Subscription" (3) |
| subscription_frequency | select | Conditional | subscription_frequency | Show if "Recurring/Subscription" (2) checked, Options: "Monthly" (1), "Yearly" (2), "Other" (3) |
| subscription_frequency_other | text | Conditional | subscription_frequency_other | Show if subscription_frequency="Other" (3), min 5 chars |

**Revenue Model Conditional Logic**:
```
IF marketingModel includes "Recurring/Subscription" (value 2):
  - SHOW subscription_frequency dropdown
  - MAKE subscription_frequency REQUIRED

  IF subscription_frequency = "Other" (value 3):
    - SHOW subscription_frequency_other text field
    - MAKE subscription_frequency_other REQUIRED
ELSE:
  - HIDE subscription_frequency dropdown
  - HIDE subscription_frequency_other text field
  - CLEAR both fields
```

**Legal Address** (6 fields - All REQUIRED, manual entry):

| Field | Type | Name Attribute | Notes |
|-------|------|---|---|
| street_number | text | street_number | REQUIRED, max 10 chars |
| street_address | text | street_address | REQUIRED, max 60 chars |
| city | text | city | REQUIRED, max 60 chars |
| state | text | state | REQUIRED, max 40 chars |
| postal_code | text | postal_code | REQUIRED, max 15 chars |
| address_country | select | address_country | REQUIRED - normal dropdown (no search), API: `/api/partner/countries` (see [reference/DROPDOWNS_STATIC_REFERENCE.md](../reference/DROPDOWNS_STATIC_REFERENCE.md)) |

**Physical/Mailing Address** (7 fields - Conditional on radio selection, manual entry):

⚠️ **STRICT INSTRUCTION**: `is_physical_address_same_as_legal_address` **must be CHECKED by default** (value='1' = same address). **Only if value=0 (unchecked), collect physical address fields.**

| Field | Type | Name Attribute | Notes |
|-------|------|---|---|
| is_physical_address_same_as_legal_address | radio | is_physical_address_same_as_legal_address | **DEFAULT CHECKED (value='1')** - Options: checked='1'(same), unchecked='0'(different) |
| physical_address_street_number | text | physical_address_street_number | **REQUIRED ONLY if value='0'** |
| physical_address_street_address | text | physical_address_street_address | **REQUIRED ONLY if value='0'** |
| physical_address_city | text | physical_address_city | **REQUIRED ONLY if value='0'** |
| physical_address_state | text | physical_address_state | **REQUIRED ONLY if value='0'** |
| physical_address_postal_code | text | physical_address_postal_code | **REQUIRED ONLY if value='0'** |
| physical_address_country | select | physical_address_country | **REQUIRED ONLY if value='0'** - normal dropdown (no search), API: `/api/partner/countries` |

---

## Pre-fill from Step 1

**legal_name and name (DBA name) fields:**
- Both fields are **pre-populated** with the Step 1 `name` value (company name entered by merchant)
- This is server-side data: `$company->name` or `$application->company->name`
- User can edit these values if needed
- Save any changes to the database on Step 2 submission

**Implementation** (server-side pre-population):
```blade
<!-- In Step 2 form (server-side render) -->
<input type="text" name="legal_name" value="{{ $company->name }}"
       placeholder="Legal Company Name" max="60" required>

<input type="text" name="name" value="{{ $company->name }}"
       placeholder="DBA (Doing Business As) Name" max="60" required>
```

**Or if client-side population**:
```javascript
// On Step 2 page load, retrieve Step 1 company name
const step1CompanyName = localStorage.getItem('step1_company_name')
                      || $('input[name="name"]').val(); // fallback

// Pre-fill Step 2 fields
$('input[name="legal_name"]').val(step1CompanyName);
$('input[name="name"]').val(step1CompanyName);
```

---

## Libraries Required

- `Cleave.js` v1.6.0 - Federal Tax ID input masking (see Federal Tax ID Input Format)
- Date picker library (e.g., Flatpickr, Bootstrap DatePicker, or similar)
- `intl-tel-input` - Phone formatting

---

## Date Picker - Business Formed Field

**Field**: `business_formed`
**Type**: DATE input with date picker
**Format**: YYYY-MM-DD
**Validation**:
- Required
- Must be on or after 1800-01-01
- Must be today or in the past (cannot be future date)

**HTML**:
```html
<div class="form-group">
    <label for="business_formed">Business Formation Date</label>
    <input type="date" id="business_formed" name="business_formed"
           class="form-control" required
           min="1800-01-01" max="today">
</div>
```

**JavaScript (Flatpickr Example)**:
```javascript
// Initialize date picker for business_formed
flatpickr('#business_formed', {
    mode: 'single',
    format: 'Y-m-d',
    minDate: '1800-01-01',
    maxDate: new Date(), // Cannot select future date
    defaultDate: null,
    placeholder: 'YYYY-MM-DD'
});
```

**Validation**: Backend validates via ApplicationStepRequest:
```php
'business_formed' => 'required|date_format:Y-m-d|after_or_equal:1800-01-01|before_or_equal:today'
```

---

**Continue to**: [STEP2_COMPANY_INFORMATION_CONDITIONALS.md](STEP2_COMPANY_INFORMATION_CONDITIONALS.md) for dependent field logic, Federal Tax ID formatting, form submission, and API integration.
