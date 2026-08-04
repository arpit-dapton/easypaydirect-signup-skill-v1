# Dependent Fields Reference — Steps 4-6

Conditional field visibility rules for Steps 4-6, plus the full summary table and implementation best practices. See [DEPENDENT_FIELDS_STEP1_3.md](DEPENDENT_FIELDS_STEP1_3.md) for Steps 1-3.

## Contents
- Step 4: Owner Information
- Step 5: Banking Information
- Step 6: Final Details
- Summary Table
- Best Practices for Implementation

---

## Step 4: Owner Information

### Ownership Percentage → Owner 2 Section

**Condition**: When owner[1][ownership_percentage] < 51

**Action**: Show and enable Owner 2 section

```html
<!-- Owner 2 Section - Hidden by Default -->
<div id="owner-section-section" style="display: none;">
    <!-- All owner 2 fields -->
</div>
```

```javascript
// Implementation
$('#owner\\[1\\]\\[ownership_percentage\\]').on('change', function() {
    const percentage = parseInt($(this).val()) || 0;

    if (percentage < 51) {
        // Show Owner 2 section
        $('#owner-section-section').show();

        // Make Owner 2 fields required
        $('#owner-section-section').find('input[required], select[required]')
            .prop('required', true);

        // Show hint
        $('#additional_owner_hint_text').show();
    } else {
        // Hide Owner 2 section
        $('#owner-section-section').hide();

        // Make Owner 2 fields optional
        $('#owner-section-section').find('input, select')
            .prop('required', false);

        // Clear Owner 2 values
        $('#owner-section-section').find('input, select').val('');
    }
});
```

### Bankruptcy Filed → Discharged Status

**Condition**: When filed_bankruptcy = 1 (Yes)

**Action**: Show bankruptcy_discharged field

```javascript
// Implementation
$('input[name="owner[1][filed_bankruptcy]"]').on('change', function() {
    if (this.value === '1') {
        $('.bankruptcy-discharged-section').show();
        $('.bankruptcy-discharged-section').find('input, select')
            .prop('disabled', false).prop('required', true);
    } else {
        $('.bankruptcy-discharged-section').hide();
        $('.bankruptcy-discharged-section').find('input, select')
            .prop('disabled', true).prop('required', false).val('');
    }
});
```

### Bankruptcy Discharged → Discharge Date

**Condition**: When bankruptcy_discharged = 1 (Yes)

**Action**: Show bankruptcy_discharged_date field

```javascript
// Implementation
$('input[name="owner[1][bankruptcy_discharged]"]').on('change', function() {
    if (this.value === '1') {
        $('.bankruptcy-date-section').show();
        $('.bankruptcy-date-section').find('input')
            .prop('disabled', false).prop('required', true);
    } else {
        $('.bankruptcy-date-section').hide();
        $('.bankruptcy-date-section').find('input')
            .prop('disabled', true).prop('required', false).val('');
    }
});
```

---

## Step 5: Banking Information

### Country → Institution Number (Canada Only)

**Condition**: When the **Step 1** `country` = `"CA"` (Canada)

> ⚠️ **Country is NOT collected on Step 5.** There is no `#country` `<select>` on this step, so do **not** bind `$('#country').on('change', …)`. Read the persisted Step 1 country value once on page load and toggle visibility. Binding to a non-existent `#country` element is why generators wrongly add a country dropdown here.

**Action**: Show institution_number field

```javascript
// Implementation — country comes from Step 1 (persisted), evaluated once on load
var companyCountry = "{{ $company->country }}"; // code/slug, e.g. "US" or "CA"
if (companyCountry === 'CA') {  // Canada
    $('.institution-number-section').show();
    $('#institution_number').prop('required', true);

    $('.currency-section').show();
    $('input[name="customer_pay_currency"]').prop('required', true);
} else {
    $('.institution-number-section').hide();
    $('#institution_number').prop('required', false);

    $('.currency-section').hide();
    $('input[name="customer_pay_currency"]').prop('required', false);
}
```

### Currency Selection (Canada Only)

**Condition**: When the **Step 1** `country` = `"CA"` (Canada)

**Action**: Show customer_pay_currency radio

```html
<!-- Hidden by default, shown for Canada -->
<div class="currency-section" style="display: none;">
    <label>
        <input type="radio" name="customer_pay_currency" value="USD" required>
        USD
    </label>
    <label>
        <input type="radio" name="customer_pay_currency" value="CAD" required>
        CAD
    </label>
</div>
```

### Currently Processing → Processor Name

**Condition**: When current_processing = 1 (Yes)

**Action**: Show processor_name field

```javascript
// Implementation
$('input[name="current_processing"]').on('change', function() {
    if (this.value === '1') {
        $('.processor-name-section').show();
        $('#processor_name').prop('required', true);
    } else {
        $('.processor-name-section').hide();
        $('#processor_name').prop('required', false).val('');
    }
});
```

---

## Step 6: Final Details

### Referral Source → Other Text

**Condition**: When howdidyouhear = "Other" OR "Friend"

**Action**: Show howdidyouhear_other field

```javascript
// Implementation
$('#howdidyouhear').on('change', function() {
    const value = $(this).val();

    if (value === 'Other' || value === 'Friend') {  // Adjust based on actual API values
        $('.referral-other-section').show();
        $('#howdidyouhear_other').prop('required', true);
    } else {
        $('.referral-other-section').hide();
        $('#howdidyouhear_other').prop('required', false).val('');
    }
});
```

### Bad Experience → Provider Name

**Condition**: When bad_experience = 1 (Yes)

**Action**: Show bad_experience_provider field

```javascript
// Implementation
$('input[name="bad_experience"]').on('change', function() {
    if (this.value === '1') {
        $('.bad-experience-section').show();
        $('#bad_experience_provider').prop('required', true);
    } else {
        $('.bad-experience-section').hide();
        $('#bad_experience_provider').prop('required', false).val('');
    }
});
```

### Terms Checkbox Validation

**Condition**: Form cannot submit without acceptance

**Action**: Require checkbox value = 1

```javascript
// jQuery Validate Rule
$('#apply-form').validate({
    rules: {
        accept_terms: {
            required: true
        }
    },
    messages: {
        accept_terms: 'You must accept the Terms & Conditions'
    }
});

// Manually validate if needed
if ($('#accept_terms').val() !== '1') {
    alert('Please accept Terms & Conditions');
    return false;
}
```

---

## Summary Table

| Step | Trigger Field | Action | Target Field | Condition |
|------|---|---|---|---|
| 1 | country | show/hide | business_state | country="US" |
| 2 | country | show/hide | business_register_number | country≠"US" |
| 2 | marketingModel | show/hide | subscription_frequency | includes "2" (Recurring) |
| 2 | subscription_frequency | show/hide | subscription_frequency_other | value="3" (Other) |
| 2 | is_physical_address_same_as_legal | show/hide | physical_address_section | value="yes" |
| 3 | fulfillment_by | show/hide | fullfillment_company | value="Vendor" or "Others" |
| 3 | shopping_cart | show/hide | shopping_cart_other | value="Other" or "API / Custom Integration" |
| 4 | ownership_percentage[1] | show/hide | owner_2_section | value<51 |
| 4 | filed_bankruptcy[1] | show/hide | bankruptcy_discharged[1] | value=1 |
| 4 | bankruptcy_discharged[1] | show/hide | bankruptcy_date[1] | value=1 |
| 5 | Step 1 country (persisted, no dropdown) | show/hide | institution_number | country="CA" (Canada) |
| 5 | Step 1 country (persisted, no dropdown) | show/hide | customer_pay_currency | country="CA" (Canada) |
| 5 | current_processing | show/hide | processor_name | value=1 |
| 6 | howdidyouhear | show/hide | howdidyouhear_other | value="Other" or "Friend" |
| 6 | bad_experience | show/hide | bad_experience_provider | value=1 |

---

**Total Dependent Fields**: 20
**Conditional Show/Hide**: 17
**Conditional Required**: 10
**Conditional Validation**: 1 (slider sum)
**Multi-Level Hierarchies**: 2 (Revenue Model → Subscription → Other, Owner 1 < 51% → Owner 2)

---

## Best Practices for Implementation

### 1. Initial State (Critical)
```html
<!-- ALWAYS start hidden, even if will show immediately -->
<div id="dependent_section" style="display: none;">
    <!-- Content here -->
</div>
```

### 2. Make Fields Optional by Default
```html
<!-- Don't require conditional fields in HTML -->
<input type="text" name="field" placeholder="...">
<!-- Only make required via JavaScript when section shown -->
```

### 3. JavaScript Toggle Pattern
```javascript
$('#trigger').on('change', function() {
    if (shouldShow(this.value)) {
        $('#dependent_section').show();
        $('#dependent_section').find('input').prop('required', true);
    } else {
        $('#dependent_section').hide();
        $('#dependent_section').find('input').prop('required', false);
    }
});
```

### 4. Form Submission Filtering
```javascript
// Remove hidden fields before API submission
function filterFormData(formSelector) {
    const form = document.querySelector(formSelector);
    const formData = new FormData(form);

    const fieldsToRemove = [];
    form.querySelectorAll('input, select, textarea').forEach(field => {
        if (!field.offsetParent) { // hidden element
            fieldsToRemove.push(field.name);
        }
    });

    fieldsToRemove.forEach(name => formData.delete(name));
    return formData;
}
```

### 5. Data Persistence
```javascript
// When loading saved data, still hide conditional sections initially
// Only show if the trigger value matches
$(document).ready(function() {
    // Load saved data
    loadSavedFormData();

    // Apply conditional logic based on loaded values
    $('#trigger').trigger('change');
});
```

---

**See also**: [DEPENDENT_FIELDS_STEP1_3.md](DEPENDENT_FIELDS_STEP1_3.md) for Steps 1-3.
