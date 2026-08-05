---
name: signup-step-2-conditionals
description: Step 2 Company Information - Dependent fields, Federal Tax ID formatting, form submission, and API integration
---

# STEP 2: Company Information — Conditionals & Submission

Continuation of [STEP2_COMPANY_INFORMATION.md](STEP2_COMPANY_INFORMATION.md) (fields, pre-fill, libraries, date picker). This file covers dependent field logic, Federal Tax ID formatting, form submission, and API integration.

---

## Dependent Fields & Conditionals

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`).

⚠️ **EXCEPTION**: `is_physical_address_same_as_legal_address` radio must be **CHECKED by default** (value='1'). Physical address fields section must be hidden by default.

### Country-Based Conditionals (Level 1)

> ℹ️ **`country` is selected in Step 1**, not here. The conditionals below read that persisted Step 1 value (server-side it is available as `$company->country`; client-side, load it into the page and compare). `country` and `business_state` themselves are **not** rendered on Step 2.

**business_register_number Field** (Show ONLY if country≠"US"):
- **Conditional Logic**:
  ```javascript
  IF country ≠ "US" (not United States):
    SHOW business_register_number field
    MAKE business_register_number REQUIRED
    maxlength = 20 (or 11 if Canada)
  ELSE:
    HIDE business_register_number field
    CLEAR value
  ```

**federal_tax_id Label** (Changes based on country):
- **If country="US"**: Label = "Federal Tax ID"
- **If country≠"US"**: Label = "Federal Tax ID (or Corporation Tax Number equivalent)"

### Industry Type Conditionals

**industry_type_other Field** (Show ONLY if industry_type="other"):
- **Conditional Logic**:
  ```javascript
  IF industry_type = "other":
    SHOW industry_type_other field
    MAKE industry_type_other REQUIRED
  ELSE:
    HIDE industry_type_other field
    CLEAR value
  ```
- **Field Details**: Text input, max 255 chars, placeholder "Please specify your industry"

---

## Federal Tax ID Input Format (REQUIRED)

The `federal_tax_id` input (`id="txn_id"`, `name="federal_tax_id"`) is a **formatted, masked input** driven by [Cleave.js](https://github.com/nosir/cleave.js) v1.6.0. It is **not** a plain text box. Set `inputmode="numeric"`, `maxlength="20"` on the input, and apply the mask below whenever `country` or `business_organized` changes.

**Mask depends on the Step 1 `country`**:

| Country | Cleave config | Result |
|---------|---------------|--------|
| US / Canada / Puerto Rico | `{ numericOnly: true, blocks: [3, 2, 4], delimiters: ['-', '-'] }` | `123-45-6789` |
| All other countries | `{ numericOnly: true, blocks: [2, 7], delimiters: ['-'] }` | `12-3456789` |

**Enable / disable rules** (evaluated on load and whenever `country` or `business_organized` changes):
- **Sole Proprietorship** (`business_organized = "Sole-Proprietorship"`): **disable** the field, remove `required`, clear the value. Show the hint: *"Sole proprietors: You will be asked for your SSN when setting up your account."*
- **Canada** (`country = "CA"`): **disable** the field, remove `required`, clear the value.
- **Any other valid country + business type**: enable, mark `required`, and (re)apply the Cleave mask for that country.
- Always **destroy the previous Cleave instance** before creating a new one (re-formatting on country change), else masks stack.

**Validation** (`jquery-validate` custom method):
- Minimum **9 digits** for US/Canada/PR.
- Minimum **11 digits** when also Sole Proprietorship (`business_organized = "Sole-Proprietorship"`).
- Message: *"Please enter at least 9 digits"*.

**Reference implementation** (uses country/business_organized **slug** values — not numeric ids):
```javascript
let activeFormat = null;

function clearActiveFormat() {
    if (activeFormat) { activeFormat.destroy(); activeFormat = null; }
}

function manageFederalTaxIdFormat(country) {
    const isUSCanadaPR = ['US', 'CA', 'PR'].includes(country);
    const formatConfig = isUSCanadaPR
        ? { numericOnly: true, blocks: [3, 2, 4], delimiters: ['-', '-'] }  // 123-45-6789
        : { numericOnly: true, blocks: [2, 7], delimiters: ['-'] };          // 12-3456789
    activeFormat = new Cleave('#txn_id', formatConfig);
}

function manageFederalTaxId(country, businessOrganized) {
    const $txnId = $('#txn_id');
    const isSoleProprietorship = businessOrganized === 'Sole-Proprietorship';
    clearActiveFormat();
    manageFederalTaxIdLabel(country); // swap label text per country (see above)

    if (isSoleProprietorship || country === 'CA') {
        // Sole proprietor or Canada -> disabled, not required, cleared
        $txnId.prop('disabled', true).removeAttr('required')
              .removeClass('is-invalid error').val('').trigger('change');
    } else {
        $txnId.prop('disabled', false).attr('required', 'required');
        manageFederalTaxIdFormat(country);
    }
}
```

**Required library** — add Cleave.js to the page:
```html
<script defer src="https://cdnjs.cloudflare.com/ajax/libs/cleave.js/1.6.0/cleave.min.js"></script>
```

### Physical Address Conditional (Level 1)

⚠️ **CRITICAL LOGIC**:
- **Default**: `is_physical_address_same_as_legal_address` = **1 (CHECKED)** = Same as legal address → **HIDE physical address fields**
- **When UNCHECKED** (value=0) = Different from legal address → **SHOW physical address fields** (REQUIRED)

**HTML - Radio Button (Default Checked)**:
```html
<div class="form-group">
    <label>Is physical address same as legal address?</label>
    <div class="custom-control custom-radio">
        <input type="radio" id="is_same_yes" name="is_physical_address_same_as_legal_address"
               value="1" class="custom-control-input" checked>
        <label class="custom-control-label" for="is_same_yes">
            Yes, same as legal address
        </label>
    </div>
    <div class="custom-control custom-radio">
        <input type="radio" id="is_same_no" name="is_physical_address_same_as_legal_address"
               value="0" class="custom-control-input">
        <label class="custom-control-label" for="is_same_no">
            No, different physical address
        </label>
    </div>
</div>

<!-- Physical address fields - HIDDEN BY DEFAULT -->
<div id="physical_address_section" style="display: none;">
    <!-- Physical address manual entry fields here -->
</div>
```

**JavaScript - Show/Hide Logic**:
```javascript
$(document).ready(function() {
    // On change, show/hide physical address section
    $('input[name="is_physical_address_same_as_legal_address"]').on('change', function() {
        const value = $(this).val();

        if (value === '0') {
            // Different address - SHOW physical address fields (REQUIRED)
            $('#physical_address_section').show();
            $('#physical_address_section').find('input, select').prop('required', true);
        } else {
            // Same address - HIDE physical address fields (NOT REQUIRED)
            $('#physical_address_section').hide();
            $('#physical_address_section').find('input, select').prop('required', false).val('');
        }
    });

    // Initialize on page load (should be hidden since default is checked='1')
    $('#physical_address_section').hide();
});
```

### Revenue Model Conditionals (Level 2)

- **subscription_frequency**: Show if marketingModel includes "Recurring/Subscription" (value 2)
- **subscription_frequency_other**: Show if subscription_frequency="Other" (value 3)

---

## Form Submission & Redirect

**On Success (HTTP 200/201)**:
```javascript
// Response example:
{
  "status": true,
  "message": "Company information saved successfully",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "step_count": 2
}

// UUID already obtained from Step 1 — pass it with this and every subsequent step
// Redirect to Step 3 (URL pattern is a suggestion; see skill.md → URL Routing & Navigation)
```

**On Validation Error (HTTP 422)**:
```javascript
// Response example:
{
  "status": false,
  "message": "Validation failed",
  "errors": {
    "company_name": ["Company name is required"],
    "legal_address_street_number": ["Street number is required"]
  }
}

// Display field errors
// DO NOT change URL - user stays on Step 2
for (const [field, messages] of Object.entries(response.errors)) {
  displayErrorForField(field, messages[0]);
}
```

---

## API Integration

**Dropdown Data**:
```
GET /api/partner/countries
GET /api/partner/industry-types
```

No authentication header required.

**Form Submission**:
```
POST /api/v1/application/step
Headers: None required (see skill.md → Authentication)
Payload:
  step_count: 2
  section: company_info
  all Step 2 fields
Filter: Remove hidden/disabled fields before submission
Response: { next_step_url, message }
```

---

## Field Summary

**Total Fields**: 26 (country + business_state + annual_sales moved to Step 1)
**Required Fields**: 21
**Conditional Fields**: 6 (business_register_number, industry_type_other, physical address fields, subscription_frequency, subscription_frequency_other)

---

**See also**: [STEP2_COMPANY_INFORMATION.md](STEP2_COMPANY_INFORMATION.md) for fields and pre-fill.
