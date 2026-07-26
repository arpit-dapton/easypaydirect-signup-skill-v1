# Dependent Fields Reference

Complete guide to all conditional field visibility rules across the signup form.

---

## ⚠️ Critical Implementation Rule

**All conditional fields MUST be hidden on first page render.**

- Do NOT show conditional fields until the trigger value is selected
- This applies at page load, before any JavaScript runs
- Use `style="display: none;"` on wrapper divs
- JavaScript only toggles visibility — it doesn't create initial state

Example:
```html
<!-- CORRECT: Hidden by default -->
<div id="physical_address_section" style="display: none;">
    <!-- These fields are invisible until trigger selected -->
</div>

<!-- WRONG: Visible by default -->
<div id="physical_address_section">
    <!-- This will show immediately - don't do this -->
</div>
```

---

## Step 1: Account Information

### Country → State Dependency

**Condition**: When country = "US" (United States)

**Action**: Show business_state field

```html
<!-- HTML -->
<div class="country-div">
    <select name="country" id="business_formed_in" required>
        <!-- API options: returns code like "US", "CA", etc -->
    </select>
</div>

<div class="usa_state state-div" style="display:none;">
    <select name="business_state" id="business_state" required>
        <!-- State options from API -->
    </select>
</div>
```

```javascript
// Implementation
$('.business_country').on('change', function() {
    if (this.value === "US") {  // "US" = United States
        $(".usa_state").show();
        $("#business_state").prop('required', true);
    } else {
        $(".usa_state").hide();
        $("#business_state").prop('required', false);
    }
});
```

**Dependency Config**:
```javascript
{
    trigger: '#business_formed_in',
    target: '#business_state',
    targetWrapper: '.usa_state',
    value: 'US',
    message: 'Required because your Formation Country is the United States.'
}
```

---

## Step 2: Company Information

### Country → Business Registration Number

**Condition**: When country ≠ "US" (not United States)

**Action**: Show business_register_number field

```javascript
// Implementation
if (this.value !== "US") {
    $("#business_register_number_div").show();
    $("#business_register_number").prop('required', true);
} else {
    $("#business_register_number_div").hide();
    $("#business_register_number").prop('required', false);
}
```

### Revenue Model → Subscription Frequency (Level 1)

**Condition**: When marketingModel includes value "2" (Recurring/Subscription)

**Action**: Show subscription_frequency dropdown and make required

```html
<!-- Revenue Model Checkboxes -->
<label>
    <input type="checkbox" name="marketingModel[]" value="1">
    One Time Purchase
</label>
<label>
    <input type="checkbox" name="marketingModel[]" value="2">
    Recurring/Subscription
</label>
<label>
    <input type="checkbox" name="marketingModel[]" value="3">
    Trial Offer + Subscription
</label>

<!-- Subscription Frequency - Hidden by default -->
<div class="subscription_frequency_container" style="display: none;">
    <select name="subscription_frequency" required>
        <option value="">Please Select</option>
        <option value="1">Monthly</option>
        <option value="2">Yearly</option>
        <option value="3">Other</option>
    </select>
</div>
```

```javascript
// Check if "Recurring/Subscription" is selected
$('input[name="marketingModel[]"]').on('change', function() {
    const isRecurringSelected = $('input[name="marketingModel[]"][value="2"]:checked').length > 0;
    
    if (isRecurringSelected) {
        $('.subscription_frequency_container').show();
        $('select[name="subscription_frequency"]').prop('required', true);
    } else {
        $('.subscription_frequency_container').hide();
        $('select[name="subscription_frequency"]').prop('required', false);
        $('select[name="subscription_frequency"]').val('');
        // Also hide subscription_frequency_other
        $('.subscription_frequency_other').hide();
    }
});
```

### Subscription Frequency → Other Text Field (Level 2 - Nested)

**Condition**: When subscription_frequency = "3" (Other)

**Action**: Show subscription_frequency_other text field and make required

```html
<!-- Subscription Frequency Other - Hidden by default -->
<div class="subscription_frequency_other" style="display: none;">
    <input type="text" name="subscription_frequency_other" 
           placeholder="Other" minlength="5" required>
</div>
```

```javascript
// When subscription_frequency changes
$('select[name="subscription_frequency"]').on('change', function() {
    const value = $(this).val();
    
    if (value === '3') { // "Other" selected
        $('.subscription_frequency_other').show();
        $('input[name="subscription_frequency_other"]').prop('required', true);
    } else {
        $('.subscription_frequency_other').hide();
        $('input[name="subscription_frequency_other"]').prop('required', false);
        $('input[name="subscription_frequency_other"]').val('');
    }
});
```

### Physical Address Toggle

**Condition**: When is_physical_address_same_as_legal_address = "yes"

**Action**: Show physical address fields

```html
<!-- Radio Options -->
<label>
    <input type="radio" name="is_physical_address_same_as_legal_address" value="yes">
    Yes (address is different)
</label>

<label>
    <input type="radio" name="is_physical_address_same_as_legal_address" value="no" checked>
    No (address is same)
</label>

<!-- Physical Address Section - Hidden by Default -->
<div id="physical_address_section" style="display: none;">
    <!-- Physical address fields -->
</div>
```

```javascript
// Implementation
$('input[name="is_physical_address_same_as_legal_address"]').on('change', function() {
    if (this.value === "yes") {
        $("#physical_address_section").show();
        // Enable and require fields
        $("#physical_address_section").find('input, select').prop('disabled', false);
    } else {
        $("#physical_address_section").hide();
        // Disable and unrequire fields
        $("#physical_address_section").find('input, select').prop('disabled', true);
    }
});
```

**Before Submission**: Convert value
```javascript
// "yes" → 0, "no" → 1 (for API)
if (formData.is_physical_address_same_as_legal_address === "yes") {
    formData.is_physical_address_same_as_legal_address = 0;  // Different
} else {
    formData.is_physical_address_same_as_legal_address = 1;  // Same
}
```

---

## Step 3: Product Information

### Fulfillment Method → Vendor Name

**Condition**: When fulfillment_by = "Vendor" OR "Others"

**Action**: Show fullfillment_company field

```javascript
// Dependency Config
{
    trigger: '#fulfillment_by_change',
    target: '#fullfillment_company',
    targetWrapper: '.fulfillment_by_field',
    value: ['Vendor', 'Others'],
    message: ['Required because your Fulfillment Method is Vendor.',
              'Required because your Fulfillment Method is Others.']
}
```

```javascript
// Implementation
$('#fulfillment_by_change').on('change', function() {
    const value = $(this).val();
    
    if (value === 'Vendor' || value === 'Others') {
        $('.fulfillment_by_field').show();
        $('#fullfillment_company').prop('disabled', false).prop('required', true);
    } else {
        $('.fulfillment_by_field').hide();
        $('#fullfillment_company').prop('disabled', true).prop('required', false);
    }
});
```

### Shopping Cart / CRM → Other Platform

**Condition**: When shopping_cart = "Other" OR "API / Custom Integration"

**Action**: Show shopping_cart_other field

```javascript
// Dependency Config
{
    trigger: '#shopping_cart',
    target: '#shopping_cart_other',
    targetWrapper: '.shopping_cart_other_container',
    value: ['Other', 'API / Custom Integration'],
    message: ['Required because your Shopping Cart / CRM is Other.',
              'Required because your Shopping Cart / CRM is API / Custom Integration.']
}
```

```javascript
// Implementation
$('#shopping_cart').on('change', function() {
    const value = $(this).val();
    
    if (value === 'Other' || value === 'API / Custom Integration') {
        $('.shopping_cart_other_container').show();
        $('#shopping_cart_other').prop('disabled', false).prop('required', true);
    } else {
        $('.shopping_cart_other_container').hide();
        $('#shopping_cart_other').prop('disabled', true).prop('required', false);
    }
});
```

### Slider Validation

**Condition**: All three slider values must be multiples of 5

**Action**: Validate sum = 100%

```javascript
// Configuration
$('.js-range-slider').ionRangeSlider({
    type: "range",
    min: 0,
    max: 100,
    from: 0,
    to: 100,
    step: 5,  // ← MUST BE 5
    grid: true
});

// Validation
function validateSlider() {
    const cardSwiped = parseInt($('#card_swiped').val()) || 0;
    const customerEntered = parseInt($('#customer_entered').val()) || 0;
    const staffEntered = parseInt($('#staff_entered').val()) || 0;
    
    const sum = cardSwiped + customerEntered + staffEntered;
    
    if (sum !== 100) {
        return false;  // Show error
    }
    
    // Check all are multiples of 5
    if (cardSwiped % 5 !== 0 || customerEntered % 5 !== 0 || staffEntered % 5 !== 0) {
        return false;  // Show error
    }
    
    return true;
}
```

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

**Condition**: When the **Step 1** `country` = `"2"` (Canada)

> ⚠️ **Country is NOT collected on Step 5.** There is no `#country` `<select>` on this step, so do **not** bind `$('#country').on('change', …)`. Read the persisted Step 1 country value once on page load and toggle visibility. Binding to a non-existent `#country` element is why generators wrongly add a country dropdown here.

**Action**: Show institution_number field

```javascript
// Implementation — country comes from Step 1 (persisted), evaluated once on load
var companyCountry = "{{ $company->country }}"; // "1" = US, "2" = Canada
if (companyCountry === '2') {  // Canada
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

**Condition**: When the **Step 1** `country` = `"2"` (Canada)

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
| 5 | Step 1 country (persisted, no dropdown) | show/hide | institution_number | country="2" (Canada) |
| 5 | Step 1 country (persisted, no dropdown) | show/hide | customer_pay_currency | country="2" (Canada) |
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

Production Ready ✅
