# Complete Dropdown Reference

All dropdown values used throughout the signup form - both from API and static options.

---

## API-Based Dropdowns

### Countries (Step 1, Step 2, Step 4, Step 5)
**Endpoint**: `GET /api/partner/countries`  
**Header**: `Authorization: {user.security_key}`  

**Response Format**:
```json
{
  "success": true,
  "data": [
    { "name": "United States", "code": "US", "id": "1" },
    { "name": "Canada", "code": "CA", "id": "2" },
    { "name": "Mexico", "code": "MX", "id": "3" }
  ]
}
```

**⚠️ CRITICAL: Use the `code` field (slug) as option value, NOT `id`**

**Form Submission**: Use code values
- United States: "US"
- Canada: "CA"
- Mexico: "MX"

**Conditional Logic**: Use code values for show/hide logic
- Show business_state if country = "US"
- Show driver license fields if country = "US"

---

### US States (Step 1, Step 5)
**Endpoint**: `GET /api/partner/states`  
**Header**: `Authorization: {user.security_key}`  
**Conditional**: Show only when country = "US" (use country code/slug, not id)

**Response Format**:
```json
{
  "success": true,
  "data": [
    { "name": "California", "code": "CA" },
    { "name": "New York", "code": "NY" },
    { "name": "Texas", "code": "TX" }
  ]
}
```

---

### Industry Types (Step 2)
**Endpoint**: `GET /api/partner/industry-types`  
**Header**: `Authorization: {user.security_key}`  

**Response Format**:
```json
{
  "success": true,
  "data": [
    { "name": "Retail", "slug": "retail" },
    { "name": "E-commerce", "slug": "ecommerce" },
    { "name": "SaaS", "slug": "saas" }
  ]
}
```

**Known Values** (Static Fallback):
```
- Retail
- E-commerce
- SaaS
- Services
- Adult-Other
- [Others from API]
```

---

### Referral Sources (Step 6)
**Endpoint**: `GET /api/partner/referral-sources`  
**Header**: `Authorization: {user.security_key}`  

**Response Format**:
```json
{
  "success": true,
  "data": [
    { "name": "Google Search", "slug": "1" },
    { "name": "Friend", "slug": "14" },
    { "name": "Live Event", "slug": "8" },
    { "name": "Other", "slug": "50" }
  ]
}
```

**Known Values**:
```
- Google Search (slug: 1)
- Friend (slug: 14)
- Live Event (slug: 8)
- Other (slug: 50)
[More from API]
```

---

### Shopping Carts / CRM (Step 3)
**Endpoint**: `GET /api/partner/shopping-carts`  
**Header**: `Authorization: {user.security_key}`  

**Response Format**:
```json
{
  "success": true,
  "data": [
    { "name": "Shopify", "slug": "1" },
    { "name": "WooCommerce", "slug": "2" },
    { "name": "Other", "slug": "8" },
    { "name": "API / Custom Integration", "slug": "58" }
  ]
}
```

**Known Values**:
```
- Shopify (id: 1)
- WooCommerce (id: 2)
- Magento
- BigCommerce
- Etsy
- Amazon
- [Others]
- Other (id: 8) ← Triggers text field
- API / Custom Integration (id: 58) ← Triggers text field
```

**Conditional Logic**:
- If value = "8" OR "58" → Show "Other Sales Platform" text field
- Otherwise → Hide text field

---

### Interest Details (Step 6)
**Endpoint**: `GET /api/partner/interest-details`  
**Header**: `Authorization: {user.security_key}`  

**Response Format**:
```json
{
  "success": true,
  "data": [
    { "name": "Capital", "slug": "capital" },
    { "name": "Resources", "slug": "resources" },
    { "name": "Marketing", "slug": "marketing" },
    { "name": "Integration", "slug": "integration" }
  ]
}
```

**Known Values**:
```
- Capital
- Resources
- Marketing
- Integration
[More from API]
```

---

## Important: Value vs Label

**All dropdowns follow this pattern:**
- **Label** (displayed to user): "0-7 days", "Full Refund", etc.
- **Value** (submitted to API): Slug format "0-7-days", "Full-Refund", etc.

Always use the **slug value** when submitting, not the display label.

---

## Google Maps Address Autocomplete Integration

### Legal Address Autocomplete

**Field**: `autocomplete` (text input, id=`autocomplete`)

**Visibility**: Shown initially ONLY if no address data exists

**Auto-populates fields**:
- `street_number` (id=`street_number`)
- `route` (id=`route`) 
- `locality` (id=`locality`)
- `administrative_area_level_1` (id=`administrative_area_level_1`)
- `country` (id=`country`, selectpicker)
- `postal_code` (id=`postal_code`)

**Behavior**: 
1. User searches and selects address from autocomplete dropdown
2. Google place_changed event fires
3. All 6 address fields populate from Google's address_components
4. Autocomplete container hides (#address_area gets d-none class)
5. Address fields container shows (#street_area removes d-none class)
6. All fields become required

**Component Mapping** (Google → Form Field):
```
street_number (short_name) → field id="street_number"
route (long_name) → field id="route"
locality (long_name) → field id="locality"
administrative_area_level_1 (short_name) → field id="administrative_area_level_1"
postal_code (short_name) → field id="postal_code"
country (long_name) → field id="country" (selectpicker)
```

### Physical Address Autocomplete

**Field**: `physical_address_autocomplete` (text input, id=`physical_address_autocomplete`)

**Visibility**: Shown ONLY if:
- Radio `is_physical_address_same_as_legal_address` = "yes" (different address)
- AND no physical address data exists yet

**Auto-populates fields**:
- `physical_address_street_number` (id=`physical_address_street_number`)
- `physical_address_street_address` (id=`physical_address_street_address`)
- `physical_address_city` (id=`physical_address_city`)
- `physical_address_state` (id=`physical_address_state`)
- `physical_address_postal_code` (id=`physical_address_postal_code`)
- `physical_address_country` (id=`physical_address_country`, selectpicker)

**Behavior**:
1. User searches and selects address from autocomplete dropdown
2. Google place_changed event fires
3. All 6 physical address fields populate from address_components
4. Autocomplete container hides (#physical_address_area gets d-none class)
5. Address fields container shows (#physical_address_street_area removes d-none class)
6. All fields become required

**Component Mapping** (Google → Form Field):
```
street_number (short_name) → field id="physical_address_street_number"
route (long_name) → field id="physical_address_street_address"
locality (long_name) → field id="physical_address_city"
administrative_area_level_1 (short_name) → field id="physical_address_state"
postal_code (short_name) → field id="physical_address_postal_code"
country (long_name) → field id="physical_address_country" (selectpicker)
```

**Key Implementation Details**:
- Both autocompletes use `types: ['geocode']` restriction
- Loop through place.address_components array
- Check component.types array against componentForm mapping
- Use short_name for state/postal, long_name for street/city/country
- Trigger 'change' and 'valid' events after populating each field
- For selectpicker country fields: call `.selectpicker('val', value)` and `.selectpicker('refresh')`
- Hide autocomplete container, show address fields container after selection

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

## Searchable Dropdowns (Live Search)

⚠️ **Only 3 dropdowns are SEARCHABLE with live search enabled:**

| Step | Field | Library | Implementation |
|------|-------|---------|-----------------|
| 1 | country | bootstrap-select | `data-live-search="true"` |
| 1 | business_state | bootstrap-select | `data-live-search="true"` |
| 3 | shopping_cart | bootstrap-select | `data-live-search="true"` |

**All other dropdowns are NORMAL (no search)**: industry_type, howdidyouhear, other_interests_capital

### Implementation with bootstrap-select (Searchable Dropdowns)

**HTML Template**:
```html
<div class="form-group">
    <label for="country">Country *</label>
    <select id="country" name="country" class="form-control selectpicker" 
            data-live-search="true" data-style="btn-light" 
            data-width="100%" required>
        <option value="" disabled selected>Select Country...</option>
        <!-- Options populated via JavaScript -->
    </select>
    <div class="invalid-feedback" id="country_error"></div>
</div>
```

**Required Libraries**:
```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap-select/1.14.0-beta3/css/bootstrap-select.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/bootstrap-select/1.14.0-beta3/js/bootstrap-select.min.js"></script>
```

**JavaScript - Fetch & Initialize**:
```javascript
$(document).ready(function() {
    const apiKey = sessionStorage.getItem('api_key');
    
    // Fetch countries for searchable dropdown
    $.ajax({
        url: '/api/partner/countries',
        type: 'GET',
        headers: { 'Authorization': apiKey },
        success: function(response) {
            const select = $('#country');
            response.data.forEach(item => {
                select.append(`<option value="${item.code}">${item.name}</option>`);
            });
            // CRITICAL: Refresh selectpicker after adding options
            select.selectpicker('refresh');
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
function validateSearchableDropdowns() {
    let valid = true;
    
    const searchables = ['country', 'business_state', 'shopping_cart'];
    
    searchables.forEach(fieldId => {
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
    if (!validateSearchableDropdowns()) {
        e.preventDefault();
        return false;
    }
});
```

### CSS for Proper UI

```css
.selectpicker {
    border: 1px solid #ced4da;
    border-radius: 0.25rem;
}

.selectpicker:focus {
    border-color: #80bdff;
    outline: 0;
    box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.selectpicker.is-invalid {
    border-color: #dc3545;
}

.selectpicker.is-invalid:focus {
    border-color: #dc3545;
    box-shadow: 0 0 0 0.2rem rgba(220, 53, 69, 0.25);
}

.bootstrap-select .dropdown-toggle:after {
    content: '';
}

.bootstrap-select .dropdown-menu {
    max-height: 400px;
    overflow-y: auto;
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

**Last Updated**: 2026-07-19  
**All API values verified** ✅
