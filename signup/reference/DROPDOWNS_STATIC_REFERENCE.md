# Static Dropdowns & Implementation Reference

Static (non-API) dropdown values, dropdown UI implementation, value conversion, and testing checklist. See [DROPDOWNS_REFERENCE.md](DROPDOWNS_REFERENCE.md) for API-backed dropdowns.

⚠️ **All dropdowns in this form are plain/normal `<select>` elements — none are searchable.** Do not use bootstrap-select, selectpicker, or any live-search library.

## Contents
- Static Dropdowns (No API Call Needed)
- Radio Button Reference
- Conversion Reference
- Dropdown Implementation
- Implementation Notes
- Testing Quick Reference

---

## Static Dropdowns (No API Call Needed)

### Job Titles (Step 4 - Both Owners)
**Field**: `title[1]` or `title[2]`

**Submit these slug values:**
```
CEO
CFO
Chairman
Co-owner
Controller
Director
General-Manager
Office-Manager
Owner
Partner
President
Treasurer
Vice-President
Other
Employee
```

---

### Fulfillment Timeframe (Step 3)
**Field**: `customer_service_time`

**Submit these slug values:**
```
0-7-days: 0-7 days (display label)
7-30-days: 7-30 days (display label)
31+days: 31+ days (display label)
```

---

### Refund Policy (Step 3)
**Field**: `refund_policy`

**Submit these slug values:**
```
Full-Refund
No-Refund
Exchange-Only
Partial-Refund
```

---

### Fulfillment Method (Step 3)
**Field**: `fulfillment_by`

**Submit these slug values:**
```
Direct-By-You
Service-Only
Vendor
Others
```

**Conditional Logic**:
- If value = "Vendor" OR "Others" → Show "Fulfillment Vendor Name" text field
- Otherwise → Hide text field

---

### Business Location Type (Step 2)
**Field**: `business_location`

**Submit these slug values:**
```
Home-Based
Co-Working
Corporate-Office
Storefront
Others
```

---

### Business Legal Structure (Step 2)
**Field**: `business_organized`

**Submit these slug values:**
```
Corporation
LLC
Partnership
Government
Sole-Proprietorship
Non-Profit
Other
```

---

### Required Deposits (Step 3)
**Field**: `leave_deposit`

```
1: Yes
0: No
```

---

### Currently Processing (Step 5)
**Field**: `current_processing`

```
1: Yes (conditionally show processor name)
0: No
```

---

### Is Physical Address Same as Legal Address? (Step 2)

**Field**: `is_physical_address_same_as_legal_address`

⚠️ **STRICT INSTRUCTION**:
- **DEFAULT**: Checked (value='1') = Same as legal address → **HIDE physical address fields**
- **When unchecked** (value='0') = Different from legal address → **SHOW physical address fields (REQUIRED)**

**HTML Values** (Radio):
```
checked='1': "Yes, same as legal address" (DEFAULT)
checked='0': "No, different physical address"
```

**API Submission Values**:
```
1: Same address (DEFAULT) - Do NOT collect/submit physical address fields
0: Different address - MUST collect and submit all physical address fields
```

**CRITICAL Logic**:
- Form loads with radio **CHECKED** (value='1')
- Physical address section is **HIDDEN by default**
- When user unchecks (value='0'), physical address fields **SHOW and become REQUIRED**
- When user checks again (value='1'), physical address fields **HIDE and become NOT REQUIRED** (clear values)

---

## Radio Button Reference

### Primary Owner Yes/No Fields (Step 4)
```
1: Yes
0: No
```

**Used for**:
- Filed Bankruptcy
- Bankruptcy Discharged
- Currently Processing (Step 5)
- Bad Experience (Step 6)
- Multiple Merchant Accounts (Step 6)

---

## Conversion Reference

### Country ID to Code
```
1 → US (United States)
2 → CA (Canada)
3 → MX (Mexico)
[Others from API]
```

### Radio/Boolean Values
```
API submission format:
1 → True / Yes
0 → False / No

HTML form values (varies):
May be: "yes"/"no", "1"/"0", true/false
Must convert before API submission!
```

---

## Dropdown Implementation

All dropdowns — including `country`, `business_state`, and `shopping_cart` — are **plain, normal `<select>` elements**. No searchable/live-search UI, and no third-party select-enhancement library is required.

**HTML Template**:
```html
<div class="form-group">
    <label for="country">Country *</label>
    <select id="country" name="country" class="form-control" required>
        <option value="" disabled selected>Select Country...</option>
        <!-- Options populated via JavaScript -->
    </select>
    <div class="invalid-feedback" id="country_error"></div>
</div>
```

**JavaScript - Fetch & Populate**:
```javascript
$(document).ready(function() {
    const apiKey = sessionStorage.getItem('api_key');

    // Fetch countries and populate the plain dropdown
    $.ajax({
        url: '/api/partner/countries',
        type: 'GET',
        headers: { 'Authorization': apiKey },
        success: function(response) {
            const select = $('#country');
            response.data.forEach(item => {
                select.append(`<option value="${item.code}">${item.name}</option>`);
            });
        }
    });

    // Validation on change
    $('#country').on('change', function() {
        const value = $(this).val();
        const errorDiv = $('#country_error');

        if (!value) {
            $(this).addClass('is-invalid');
            errorDiv.text('Country is required').show();
            return false;
        } else {
            $(this).removeClass('is-invalid');
            errorDiv.hide();

            // Trigger conditional field updates (e.g., business_state)
            updateDependentFields();
            return true;
        }
    });
});
```

**Validation on Form Submit**:
```javascript
function validateDropdowns() {
    let valid = true;

    const requiredDropdowns = ['country', 'business_state', 'shopping_cart'];

    requiredDropdowns.forEach(fieldId => {
        const select = $(`#${fieldId}`);
        const value = select.val();
        const errorDiv = $(`#${fieldId}_error`);

        if (!value || value === '') {
            select.addClass('is-invalid');
            errorDiv.text('This field is required').show();
            valid = false;
        } else {
            select.removeClass('is-invalid');
            errorDiv.hide();
        }
    });

    return valid;
}

// Call before form submission
$('#signupForm').on('submit', function(e) {
    if (!validateDropdowns()) {
        e.preventDefault();
        return false;
    }
});
```

### CSS for Proper UI

```css
.form-control.is-invalid {
    border-color: #dc3545;
}

.form-control.is-invalid:focus {
    border-color: #dc3545;
    box-shadow: 0 0 0 0.2rem rgba(220, 53, 69, 0.25);
}

.invalid-feedback {
    display: none;
    color: #dc3545;
    font-size: 0.875rem;
    margin-top: 0.25rem;
}
```

---

## Implementation Notes

### Pulling from API

All dropdown data should be fetched on form initialization:

```javascript
async function loadDropdownData() {
    const apiKey = sessionStorage.getItem('api_key');

    // Load countries
    const countries = await fetch('/api/partner/countries', {
        headers: { 'Authorization': apiKey }
    }).then(r => r.json());

    // Populate select
    const select = document.querySelector('select[name="country"]');
    countries.data.forEach(country => {
        const option = document.createElement('option');
        option.value = country.code;  // ← Use CODE/slug (US, CA, MX), NOT id!
        option.textContent = country.name;
        select.appendChild(option);
    });
}
```

### Conditional Display Logic

```javascript
// Show/hide based on dropdown value
$('#shopping_cart').on('change', function() {
    const value = $(this).val();

    if (value === '8' || value === '58') {
        $('#shopping_cart_other').show();
    } else {
        $('#shopping_cart_other').hide();
    }
});
```

### Value Conversion Before Submission

```javascript
// Convert radio values to API format
// is_physical_address_same_as_legal_address: 1=same, 0=different
function prepareFormData(formData) {
    const value = $('input[name="is_physical_address_same_as_legal_address"]:checked').val();

    if (value === '1') {
        // Checked = Same as legal address
        formData.is_physical_address_same_as_legal_address = 1;
        // Do NOT include physical address fields
        delete formData.physical_address_street_number;
        delete formData.physical_address_street_address;
        delete formData.physical_address_city;
        delete formData.physical_address_state;
        delete formData.physical_address_postal_code;
        delete formData.physical_address_country;
    } else if (value === '0') {
        // Unchecked = Different from legal address
        formData.is_physical_address_same_as_legal_address = 0;
        // MUST include all physical address fields
        // Validation will ensure they are not empty
    }

    // Boolean 0/1 values
    formData.leave_deposit = formData.leave_deposit === '1' ? 1 : 0;
    formData.current_processing = formData.current_processing === '1' ? 1 : 0;
    formData.bad_experience = formData.bad_experience === '1' ? 1 : 0;
    formData.multiple_merchant_accounts = formData.multiple_merchant_accounts === '1' ? 1 : 0;
    formData.bankruptcy_discharged = formData.bankruptcy_discharged === '1' ? 1 : 0;

    return formData;
}
```

---

## Testing Quick Reference

### Verify All Dropdowns Are Loading

```
Step 1:
  [ ] Country dropdown has 150+ countries (using code/slug values: US, CA, etc.)
  [ ] State dropdown shows only when country="US"

Step 2:
  [ ] Industry Type has options

Step 3:
  [ ] Shopping Cart has 10+ options (including "Other")

Step 6:
  [ ] How did you hear has options
  [ ] Interest Areas has options
```

### Verify Conditional Logic

```
Step 1:
  [ ] Select US → State appears
  [ ] Select Canada → State hidden

Step 3:
  [ ] Select Vendor → Vendor name field appears
  [ ] Select Direct → Vendor name field hidden
  [ ] Select "Other" shopping cart → Text field appears
  [ ] Select "Shopify" → Text field hidden

Step 2:
  [ ] Select "Yes" for different → Physical address fields appear
  [ ] Select "No" for same → Physical address fields hidden

Step 5:
  [ ] Select "Yes" for processing → Processor name field appears
  [ ] Select "No" → Processor name field hidden
```

---

**See also**: [DROPDOWNS_REFERENCE.md](DROPDOWNS_REFERENCE.md) for API-backed dropdowns.
