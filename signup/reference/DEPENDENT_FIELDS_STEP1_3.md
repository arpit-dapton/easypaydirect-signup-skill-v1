# Dependent Fields Reference — Steps 1-3

Conditional field visibility rules for Steps 1-3. See [DEPENDENT_FIELDS_STEP4_6.md](DEPENDENT_FIELDS_STEP4_6.md) for Steps 4-6, the summary table, and implementation best practices.

## Contents
- Critical Implementation Rule
- Step 1: Account Information
- Step 2: Company Information
- Step 3: Product Information

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

**Continue to**: [DEPENDENT_FIELDS_STEP4_6.md](DEPENDENT_FIELDS_STEP4_6.md) for Steps 4-6, the summary table, and implementation best practices.
