---
name: signup-step-2
description: Step 2 Company Information - Business details, legal address, revenue model, and Google Maps autocomplete
---

# STEP 2: Company Information

Second step of the 6-step signup form. Collects company business details, addresses, and revenue model information.

---

## Fields

**Company Details** (9 fields):

> ℹ️ **`country` and `business_state` are Step 1 fields** (see [STEP1_ACCOUNT_INFORMATION.md](STEP1_ACCOUNT_INFORMATION.md)). They are **NOT** collected in Step 2. The Step 2 fields below that vary by country (`federal_tax_id`, `business_register_number`) read the `country` value the merchant selected in Step 1, which persists on the application/company record.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| legal_name | text | Yes | Max 60 chars. **Pre-fill from Step 1 `name` field** |
| name | text | Yes | DBA name, max 60 chars. **Pre-fill from Step 1 `name` field** |
| industry_type | select | Yes | API: `/api/partner/industry-types` |
| industry_type_other | text | Conditional | Show if industry_type="other", max 255 chars |
| customer_service_telephone_number | tel | Yes | Use intl-tel-input |
| business_location | select | Yes | Static (slug): Home-Based, Co-Working, Corporate-Office, Storefront, Others |
| business_formed | date | Yes | YYYY-MM-DD, use date picker |
| business_organized | select | Yes | Static (slug): Corporation, LLC, Partnership, Government, Sole-Proprietorship, Non-Profit, Other |
| federal_tax_id | text | Yes | Encrypted, max 20 chars. Formatted input — see **Federal Tax ID Input Format** below. Label/format vary by Step 1 `country` |
| business_register_number | text | Conditional | Show ONLY if Step 1 `country`≠"1" (not US), max 20 chars (11 for Canada) |

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

**Legal Address** (7 fields - All REQUIRED):

| Field | Type | Name Attribute | Notes |
|-------|------|---|---|
| autocomplete | text | autocomplete | Google Maps Places autocomplete - shown initially if no address data exists |
| street_number | text | street_number | REQUIRED - Auto-populated from autocomplete, max 10 chars |
| street_address | text | route | REQUIRED - Auto-populated from autocomplete, max 60 chars |
| city | text | locality | REQUIRED - Auto-populated from autocomplete, max 60 chars |
| state | text | administrative_area_level_1 | REQUIRED - Auto-populated from autocomplete, max 40 chars |
| postal_code | text | postal_code | REQUIRED - Auto-populated from autocomplete, max 15 chars |
| country | select | country | REQUIRED - Auto-populated from autocomplete |

**Legal Address Autocomplete Behavior**:
- **Initially shown**: If no address data exists
- **On selection**: User searches and selects address from Google autocomplete
- **Auto-population**: All 6 address fields populate from Google's address_components
- **After selection**: Autocomplete hides, address fields display and become required

**Component Mapping** (Google address_components → Form fields):
- `street_number` (short_name) → field id=`street_number`
- `route` (long_name) → field id=`route`
- `locality` (long_name) → field id=`locality`
- `administrative_area_level_1` (short_name) → field id=`administrative_area_level_1`
- `postal_code` (short_name) → field id=`postal_code`
- `country` (long_name) → field id=`country` (selectpicker with refresh)

**Physical/Mailing Address** (8 fields - Conditional on radio selection):

| Field | Type | Name Attribute | Notes |
|-------|------|---|---|
| is_physical_address_same_as_legal_address | radio | is_physical_address_same_as_legal_address | Options: yes=different, no=same. Default: no |
| physical_address_autocomplete | text | physical_address_autocomplete | Google Maps autocomplete - shown only if radio="yes" AND no address data exists |
| physical_address_street_number | text | physical_address_street_number | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_street_address | text | physical_address_street_address | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_city | text | physical_address_city | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_state | text | physical_address_state | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_postal_code | text | physical_address_postal_code | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_country | select | physical_address_country | REQUIRED (when visible) - Auto-populated from autocomplete |

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

- `Google Maps Places API` - Address autocomplete (legal + physical)
- `bootstrap-select` - Country selectpicker used by the autocomplete population
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

## Google Maps Implementation

> 🚨 **MUST IMPLEMENT — do NOT skip the autocomplete input.** Generators frequently render the six address fields (`street_number`, `route`, `locality`, …) but omit the `#autocomplete` search box and its wrapper, because earlier sections describe the autocomplete as *behavior* rather than *markup*. The address box will not appear unless you render the exact HTML in **Required Address HTML** below. The JavaScript alone does nothing without this input element in the DOM.

### Why this gets skipped (and how to avoid it)

The autocomplete is **three concrete pieces that must all be present**, in this order:
1. **The Maps script tag** — loaded synchronously (no `async`/`defer`) *before* the init script, with a real API key (`{{ config('app.google_map_key') }}`), not a literal placeholder. If `google` is undefined, `google.maps.event.addDomListener(...)` throws on line 1 and the whole init aborts.
2. **The HTML** — the `#autocomplete` search input inside `#address_area`, plus the `#street_area` container (initially `d-none`) holding the six address fields. Missing input = no search box.
3. **The init JS** — binds `google.maps.places.Autocomplete` to `#autocomplete` and, on `place_changed`, fills the fields, shows `#street_area`, hides `#address_area`.

If any one is missing the feature silently does nothing — so render **all three**.

### Required Address HTML (render exactly)

```html
<!-- 1) Autocomplete search box: shown initially when no address data exists -->
<div class="form-group" id="address_area">
    <label for="autocomplete">Address Lookup</label>
    <input autocomplete="off" type="search" name="autocomplete" id="autocomplete"
           class="form-control" placeholder="Must be a physical location">
</div>

<!-- 2) Address fields: hidden until an address is picked (d-none), then shown + required -->
<div class="row d-none" id="street_area">
    <div class="form-group"><label for="street_number">Street Number</label>
        <input id="street_number" name="street_number" class="form-control"></div>
    <div class="form-group"><label for="route">Street Address</label>
        <input id="route" name="street_address" class="form-control"></div>
    <div class="form-group" id="city_area"><label for="locality">City</label>
        <input id="locality" name="city" class="form-control"></div>
    <div class="form-group" id="state_area"><label for="administrative_area_level_1">State/Province</label>
        <input id="administrative_area_level_1" name="state" class="form-control"></div>
    <div class="form-group" id="postal_area"><label for="postal_code">Postal code</label>
        <input id="postal_code" name="postal_code" class="form-control"></div>
    <div class="form-group company-address-country"><label for="country">Country</label>
        <select id="country" name="address_country" class="form-control selectpicker" data-live-search="true"></select></div>
</div>
```

> ⚠️ **ID vs name mismatch is intentional** — Google's `place_changed` handler looks up elements by the **Google component id** (`route`, `locality`, `administrative_area_level_1`, `postal_code`, `country`), while form submission uses the **`name`** (`street_address`, `city`, `state`, `postal_code`, `address_country`). Keep both exactly as shown or auto-population breaks. The physical-address block mirrors this with `physical_address_*` ids.

**Script Include** (synchronous, before init — see note above):
```html
<script src="https://maps.google.com/maps/api/js?key={{ config('app.google_map_key') }}&libraries=places" type="text/javascript"></script>
```

**Complete JavaScript Implementation**:
```javascript
var componentForm = {
    street_number: 'short_name',
    route: 'long_name',
    locality: 'long_name',
    administrative_area_level_1: 'short_name',
    country: 'long_name',
    postal_code: 'short_name'
};

google.maps.event.addDomListener(window, 'load', initialize);

function initialize() {
    // Legal Address Autocomplete (id="autocomplete")
    var input = document.getElementById('autocomplete');
    var autocomplete = new google.maps.places.Autocomplete(input);
    autocomplete.addListener('place_changed', function () {
        var place = autocomplete.getPlace();

        // Clear all fields first
        for (var component in componentForm) {
            document.getElementById(component).value = '';
            $(document.getElementById(component)).trigger('change');
            document.getElementById(component).disabled = false;
        }

        // Fill in address components
        for (var i = 0; i < place.address_components.length; i++) {
            var component = place.address_components[i];
            var addressType = null;
            
            for (var j = 0; j < component.types.length; j++) {
                if (componentForm[component.types[j]]) {
                    addressType = component.types[j];
                    break;
                }
            }
            
            if (addressType) {
                var val = component[componentForm[addressType]];
                if (addressType === 'country') {
                    // Special handling for selectpicker
                    $('#country').selectpicker('val', val);
                    $('#country').selectpicker('refresh');
                    $(document.getElementById('country')).trigger('change');
                    if ($('#country').valid()) {
                        $('.company-address-country .custom-select').removeClass('is-invalid');
                    }
                } else {
                    document.getElementById(addressType).value = val;
                    $(document.getElementById(addressType)).trigger('change');
                    $(document.getElementById(addressType)).valid();
                }
            }
        }
        
        // Show individual address fields, hide autocomplete
        $("#street_area").removeClass("d-none");
        $("#city_area").removeClass("d-none");
        $("#state_area").removeClass("d-none");
        $("#country_area").removeClass("d-none");
        $("#postal_area").removeClass("d-none");
        $("#address_area").addClass("d-none");
        $("#street_area input").attr("required", "required");
        $("#city_area input").attr("required", "required");
        $("#state_area input").attr("required", "required");
        $("#postal_area input").attr("required", "required");
        $("#address_area input").removeAttr("required");
    });
}

// Call this function when physical address radio is changed to "yes" (different address)
function initializePhysicalAddressAutocomplete() {
    var physicalInput = document.getElementById('physical_address_autocomplete');
    if (!physicalInput) return;
    
    var physicalAutocomplete = new google.maps.places.Autocomplete(physicalInput);
    var physicalComponentForm = {
        street_number: 'short_name',
        route: 'long_name',
        locality: 'long_name',
        administrative_area_level_1: 'short_name',
        country: 'long_name',
        postal_code: 'short_name'
    };

    physicalAutocomplete.addListener('place_changed', function () {
        var place = physicalAutocomplete.getPlace();
        
        // Clear all physical address fields
        var physicalFields = ['physical_address_street_number', 'physical_address_street_address', 'physical_address_city', 'physical_address_state', 'physical_address_postal_code'];
        physicalFields.forEach(function(fieldId) {
            var field = document.getElementById(fieldId);
            if (field) {
                field.value = '';
                $(field).trigger('change');
            }
        });

        // Fill in physical address components
        for (var i = 0; i < place.address_components.length; i++) {
            var component = place.address_components[i];
            var addressType = null;
            
            for (var j = 0; j < component.types.length; j++) {
                if (physicalComponentForm[component.types[j]]) {
                    addressType = component.types[j];
                    break;
                }
            }
            
            if (addressType) {
                var val = component[physicalComponentForm[addressType]];
                var fieldId = '';
                
                switch(addressType) {
                    case 'street_number':
                        fieldId = 'physical_address_street_number';
                        break;
                    case 'route':
                        fieldId = 'physical_address_street_address';
                        break;
                    case 'locality':
                        fieldId = 'physical_address_city';
                        break;
                    case 'administrative_area_level_1':
                        fieldId = 'physical_address_state';
                        break;
                    case 'postal_code':
                        fieldId = 'physical_address_postal_code';
                        break;
                    case 'country':
                        fieldId = 'physical_address_country';
                        break;
                }
                
                if (fieldId) {
                    var fieldElement = document.getElementById(fieldId);
                    if (fieldElement) {
                        if (addressType === 'country') {
                            var $countrySelect = $('#' + fieldId);
                            $countrySelect.selectpicker('val', val);
                            $countrySelect.selectpicker('refresh');
                            $countrySelect.removeClass('is-invalid error');
                            $('.physical-address-country .custom-select').removeClass('is-invalid error');
                            $countrySelect.trigger('change');
                        } else {
                            fieldElement.value = val;
                            $(fieldElement).trigger('change');
                            $(fieldElement).valid();
                        }
                    }
                }
            }
        }

        // Show individual fields, hide autocomplete
        $("#physical_address_street_area").removeClass("d-none");
        $("#physical_address_street_area input").attr("required", "required");
        $("#physical_address_city_area input").attr("required", "required");
        $("#physical_address_state_area input").attr("required", "required");
        $("#physical_address_postal_code_area input").attr("required", "required");
        $("#physical_address_area").addClass("d-none");
        $("#physical_address_area input").removeAttr("required");
    });
}

// Initialize physical address autocomplete when radio changes to "yes"
$(document).ready(function() {
    $('#is_physical_address_same_as_legal_address').change(function() {
        if ($(this).val() === 'yes') {
            initializePhysicalAddressAutocomplete();
        }
    });
});
```

**Input Field IDs** (must match HTML):
- Legal Address: `id="autocomplete"`
- Physical Address: `id="physical_address_autocomplete"`

**Output Field IDs** (where data gets populated):

Legal Address:
- `id="street_number"` ← street_number component
- `id="route"` ← route component  
- `id="locality"` ← locality component
- `id="administrative_area_level_1"` ← state component
- `id="postal_code"` ← postal code component
- `id="country"` ← country selectpicker

Physical Address:
- `id="physical_address_street_number"` ← street_number component
- `id="physical_address_street_address"` ← route component
- `id="physical_address_city"` ← locality component
- `id="physical_address_state"` ← administrative_area_level_1 component
- `id="physical_address_postal_code"` ← postal code component
- `id="physical_address_country"` ← country selectpicker

**Container IDs** (for show/hide logic):
- Legal address container: `id="address_area"` (hidden after autocomplete)
- Legal fields container: `id="street_area"`, `id="city_area"`, `id="state_area"`, `id="country_area"`, `id="postal_area"` (shown after autocomplete)
- Physical address container: `id="physical_address_area"` (hidden after autocomplete)
- Physical fields container: `id="physical_address_street_area"` (shown after autocomplete)

---

## Dependent Fields & Conditionals

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`).

### Country-Based Conditionals (Level 1)

> ℹ️ **`country` is selected in Step 1**, not here. The conditionals below read that persisted Step 1 value (server-side it is available as `$company->country`; client-side, load it into the page and compare). `country` and `business_state` themselves are **not** rendered on Step 2.

**business_register_number Field** (Show ONLY if country≠"1"):
- **Conditional Logic**:
  ```javascript
  IF country ≠ "1" (not United States):
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
- **Canada** (`country = "Canada"`): **disable** the field, remove `required`, clear the value.
- **Any other valid country + business type**: enable, mark `required`, and (re)apply the Cleave mask for that country.
- Always **destroy the previous Cleave instance** before creating a new one (re-formatting on country change), else masks stack.

**Validation** (`jquery-validate` custom method):
- Minimum **9 digits** for US/Canada/PR.
- Minimum **11 digits** when also Sole Proprietorship (`business_organized = "Sole-Proprietorship"`).
- Message: *"Please enter at least 9 digits"*.

**Reference implementation**:
```javascript
let activeFormat = null;

function clearActiveFormat() {
    if (activeFormat) { activeFormat.destroy(); activeFormat = null; }
}

function manageFederalTaxIdFormat(country) {
    const isUSCanadaPR = ['1', '2', '3'].includes(country);
    const formatConfig = isUSCanadaPR
        ? { numericOnly: true, blocks: [3, 2, 4], delimiters: ['-', '-'] }  // 123-45-6789
        : { numericOnly: true, blocks: [2, 7], delimiters: ['-'] };          // 12-3456789
    activeFormat = new Cleave('#txn_id', formatConfig);
}

function manageFederalTaxId(country, businessOrganized) {
    const $txnId = $('#txn_id');
    const isSoleProprietorship = businessOrganized === '5';
    clearActiveFormat();
    manageFederalTaxIdLabel(country); // swap label text per country (see above)

    if (isSoleProprietorship || country === '2') {
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

### Address Conditionals (Level 1)

- **Physical address section**: Show if is_physical_address_same_as_legal_address="yes"

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

// UUID already stored from Step 1, retrieve and use
const uuid = localStorage.getItem('signup_uuid');

// Redirect to Step 3
window.location.href = `/signup/step/3/${uuid}`;
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

Header: `Authorization: {user.security_key}`

**Form Submission**:
```
POST /api/v1/application/step
Headers: X-API-Key: {user.security_key}
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
**Google Maps Autocomplete**: 2 (legal + physical)

---

**Production Ready** ✅
