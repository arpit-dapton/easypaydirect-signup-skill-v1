---
name: signup-step-3
description: Step 3 Product Information - Fulfillment details, transaction methods, and product descriptions
---

# STEP 3: Product Information

Third step of the 6-step signup form. Collects product fulfillment details, transaction entry methods, and service descriptions.

---

## Fields

**Fulfillment Details** (7 fields):

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| customer_service_time | select | Yes | Normal dropdown (no search) - Static: slug values "0-7-days", "7-30-days", "31+days" |
| refund_policy | select | Yes | Normal dropdown (no search) - Static (slug): Full-Refund, No-Refund, Exchange-Only, Partial-Refund |
| fulfillment_by | select | Yes | Normal dropdown (no search) - Static (slug): Direct-By-You, Service-Only, Vendor, Others |
| fullfillment_company | text | Conditional | Show if fulfillment_by="Vendor" or "Others" |
| shopping_cart | select | Yes | Normal dropdown (no search) - API: `/api/partner/shopping-carts` |
| shopping_cart_other | text | Conditional | Show if shopping_cart="Other" or "API / Custom Integration" |
| leave_deposit | select | Yes | Options: Yes(1), No(0) |

**Transaction Entry Methods** (ONE slider control for 3 values):

⚠️ **CRITICAL: Use ONE interactive slider/control for all three values, NOT three separate sliders**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| card_swiped | number | Yes | Part of single 3-way slider: multiples of 5 (0-100%) |
| customer_entered | number | Yes | Part of single 3-way slider: multiples of 5 (0-100%) |
| staff_entered | number | Yes | Part of single 3-way slider: multiples of 5 (0-100%) |

**Validation**: Sum of all three must equal 100%

**Implementation**: Single three-way slider control
- User adjusts one value → other values auto-adjust to maintain 100% total
- All three hidden `<input>` fields store the calculated values
- Display shows: "Card Swiped: 30% | Customer Entered: 50% | Staff Entered: 20%"

**Product Details** (4 fields):

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| average_transaction_amount | text | Yes | Numeric, USD |
| highest_transaction_amount | text | Yes | Numeric, USD |
| describe_services | textarea | Yes | Min 15 chars, max 1500 |
| describe_highest_transaction | textarea | Yes | Min 15 chars, max 1500 |

---

## Libraries Required

- Standard HTML5 `<input type="range">` - Two inputs for dual-handle slider (no external library needed)
- OR `noUiSlider` / `ion-rangeslider` - For advanced styling and features (optional)
- `dependent-fields.js` - Conditional field visibility

---

## Transaction Entry Methods - Dual-Handle Range Slider

⚠️ **Must implement as ONE dual-handle range slider (user adjusts from both ends)**

**User adjusts two handles on a single slider track to distribute 100% across three transaction methods:**
- Left handle: Adjusts "Card Swiped" percentage
- Right handle: Adjusts "Card-Not Present" percentage  
- Middle (remaining): Automatically calculated as remaining percentage

### HTML Structure

```html
<div class="form-group">
    <label>How do customers enter transactions?</label>
    <p class="text-muted">Drag both ends of the slider to adjust the distribution (total = 100%)</p>
    
    <!-- Dual-handle range slider -->
    <div id="transaction_slider" class="range-slider-container" style="margin: 40px 0;">
        <!-- Slider track with two handles -->
        <input type="range" id="slider_min" class="range-slider-min" 
               min="0" max="100" step="5" value="0">
        <input type="range" id="slider_max" class="range-slider-max" 
               min="0" max="100" step="5" value="100">
        <div class="range-slider-track"></div>
    </div>
    
    <!-- Labels showing current distribution -->
    <div class="row mt-4">
        <div class="col-md-4 text-center">
            <p class="font-weight-bold">Card Swiped (card-present)</p>
            <p class="display-4" id="card_swiped_display">0%</p>
        </div>
        <div class="col-md-4 text-center">
            <p class="font-weight-bold">Customer Entered (card-not present)</p>
            <p class="display-4" id="customer_entered_display">100%</p>
        </div>
        <div class="col-md-4 text-center">
            <p class="font-weight-bold">Mail/Phone (card-not present)</p>
            <p class="display-4" id="staff_entered_display">0%</p>
        </div>
    </div>
    
    <!-- Hidden inputs for form submission -->
    <input type="hidden" name="card_swiped" id="card_swiped" value="0">
    <input type="hidden" name="customer_entered" id="customer_entered" value="100">
    <input type="hidden" name="staff_entered" id="staff_entered" value="0">
</div>

<!-- CSS for slider styling -->
<style>
.range-slider-container {
    position: relative;
    height: 50px;
}

.range-slider-min, .range-slider-max {
    position: absolute;
    width: 100%;
    height: 5px;
    background: transparent;
    pointer-events: none;
    -webkit-appearance: none;
    appearance: none;
    top: 22px;
    z-index: 5;
}

.range-slider-min::-webkit-slider-thumb, .range-slider-max::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: #27ae60;
    cursor: pointer;
    border: 2px solid white;
    box-shadow: 0 2px 4px rgba(0,0,0,0.2);
    pointer-events: all;
    z-index: 5;
}

.range-slider-min::-moz-range-thumb, .range-slider-max::-moz-range-thumb {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: #27ae60;
    cursor: pointer;
    border: 2px solid white;
    box-shadow: 0 2px 4px rgba(0,0,0,0.2);
    pointer-events: all;
}

.range-slider-track {
    position: absolute;
    height: 5px;
    background: #ecf0f1;
    border-radius: 5px;
    top: 22px;
    width: 100%;
    z-index: 1;
}

.range-slider-track::before {
    content: '';
    position: absolute;
    height: 100%;
    background: #27ae60;
    border-radius: 5px;
}

/* Slider value labels -->
.slider-labels {
    display: flex;
    justify-content: space-between;
    margin-top: 10px;
    font-size: 12px;
}

.slider-labels span {
    color: #666;
}
</style>
```

### JavaScript - Dual-Handle Range Slider

```javascript
$(document).ready(function() {
    const minSlider = $('#slider_min');
    const maxSlider = $('#slider_max');
    
    // Update on input
    minSlider.on('input', function() {
        updateSlider();
    });
    
    maxSlider.on('input', function() {
        updateSlider();
    });
    
    function updateSlider() {
        let minVal = parseInt(minSlider.val());
        let maxVal = parseInt(maxSlider.val());
        
        // Prevent handles crossing
        if (minVal > maxVal) {
            [minVal, maxVal] = [maxVal, minVal];
            minSlider.val(minVal);
            maxSlider.val(maxVal);
        }
        
        // Calculate percentages
        const cardSwiped = minVal;
        const cardNotPresent = maxVal - minVal; // Middle section
        const staffEntered = 100 - maxVal;
        
        // Update display
        $('#card_swiped_display').text(cardSwiped + '%');
        $('#customer_entered_display').text(cardNotPresent + '%');
        $('#staff_entered_display').text(staffEntered + '%');
        
        // Update hidden inputs
        $('#card_swiped').val(cardSwiped);
        $('#customer_entered').val(cardNotPresent);
        $('#staff_entered').val(staffEntered);
        
        // Update track color
        const trackPercent = ((minVal + maxVal) / 2);
        const trackBefore = $('.range-slider-track::before');
        // Visual feedback of selected range
    }
    
    // Initialize
    updateSlider();
});
```

### How It Works

```
User adjusts slider handles:

Initial state (left=0, right=100):
[●——————————————————————●]
 0     Card Swiped: 0%   100
             ↓
   Customer Entered: 100%
             ↓
   Staff Entered: 0%

User drags left handle to 30:
     [———●——————————————●]
     0  30              100
              ↓
   Card Swiped: 30%
   Customer Entered: 70%
   Staff Entered: 0%

User drags right handle to 80:
     [———●————————●——————]
     0  30       80       100
              ↓
   Card Swiped: 30%
   Customer Entered: 50% (80-30)
   Staff Entered: 20% (100-80)
```

---

## Dependent Fields

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`).

- **fullfillment_company**: Show if fulfillment_by = "Vendor" or "Others"
- **shopping_cart_other**: Show if shopping_cart = "Other" or "API / Custom Integration"
- **Three-way slider validation**: Sum of all three must equal 100% (enforced in JavaScript)

---

## Form Submission & Redirect

**On Success (HTTP 200/201)**:
```javascript
// Response example:
{
  "status": true,
  "message": "Product information saved successfully",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "step_count": 3
}

// UUID already stored from Step 1, retrieve and use
const uuid = localStorage.getItem('signup_uuid');

// Redirect to Step 4
window.location.href = `/signup/step/4/${uuid}`;
```

**On Validation Error (HTTP 422)**:
```javascript
// Response example:
{
  "status": false,
  "message": "Validation failed",
  "errors": {
    "card_swiped": ["Total transaction methods must equal 100%"],
    "describe_services": ["Description must be at least 15 characters"]
  }
}

// Display field errors
// DO NOT change URL - user stays on Step 3
for (const [field, messages] of Object.entries(response.errors)) {
  displayErrorForField(field, messages[0]);
}
```

---

## API Integration

**Dropdown Data**:
```
GET /api/partner/shopping-carts
```

Header: `Authorization: {user.security_key}`

**Form Submission**:
```
POST /api/v1/application/step
Headers: X-API-Key: {user.security_key}
Payload:
  step_count: 3
  section: product_info
  all Step 3 fields
Filter: Remove hidden/disabled fields
Response: { next_step_url, message }
```

---

## Field Summary

**Total Fields**: 13  
**Required Fields**: 10  
**Conditional Fields**: 3  
**Transaction Entry Control**: Dual-handle range slider for 3 values
  - Left handle: card_swiped (0-100%)
  - Middle section: customer_entered (auto-calculated)
  - Right handle: staff_entered position (auto-calculated)
  - Total always = 100%

---

**Production Ready** ✅
