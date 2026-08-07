---
name: signup-step-6
description: Step 6 Final Details - Referral sources, interests, and terms acceptance
---

# STEP 6: Final Details

Sixth step of the 6-step signup form. Collects referral information, interest details, and terms acceptance.

---

## Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| howdidyouhear | select | Yes | Normal dropdown (no search) - API: `/api/partner/referral-sources` |
| howdidyouhear_other | text | Conditional | Show if howdidyouhear="Other" or "Friend", max 300 chars |
| multiple_merchant_accounts | select | Yes | Normal dropdown (no search) - Static: Yes(1), No(0) |
| **transaction_device** | **select** | **No** | **Normal dropdown (no search) - Hardcoded Static Options** - See details below |
| bad_experience | select | Yes | Normal dropdown (no search) - Static: Yes(1), No(0) |
| bad_experience_happened | text | Conditional | Show if bad_experience=1, max 300 chars |
| other_interests_capital | checkbox array | No | Normal (no search) - API: `/api/partner/interest-details`, grouped checkboxes. ⚠️ Confirm this with product before building: the backend currently allows zero selections, but production's older form requires ≥1. If that requirement should carry forward, this needs to flip to required and the backend rule tightened back to match — don't assume optional is settled. |
| **terms_and_conditions_text** | **textarea** | **N/A — display only** | **Read-only/disabled, not submitted to the API** — See "Terms & Conditions Display" below |
| terms_and_conditions_agreed | checkbox | Yes | Required to submit, value must be 1 |

---

## Terms & Conditions Display (Read-Only Textarea)

Directly **above** the `terms_and_conditions_agreed` checkbox, render a read-only/disabled textarea showing the full Terms & Conditions text so the merchant can read it before agreeing.

**Content source**: The exact text to display is captured in [../reference/TERMS_AND_CONDITIONS_TEXT.md](../reference/TERMS_AND_CONDITIONS_TEXT.md) — use that file as the content source.

⚠️ **This field is display-only — it must NOT be included in the `/api/v1/application/step` payload.** Use `disabled` (not just `readonly`) so the browser excludes it from form submission automatically; if you use `readonly` instead, you must explicitly omit it from the request before sending.

**HTML**:
```html
<div class="form-group">
    <label for="terms_and_conditions_text">Terms and Conditions</label>
    <textarea id="terms_and_conditions_text"
              class="form-control"
              rows="10"
              disabled>{{-- content from reference/TERMS_AND_CONDITIONS_TEXT.md --}}</textarea>
</div>

<!-- terms_and_conditions_agreed checkbox follows immediately after -->
<div class="form-check">
    <input type="checkbox" name="terms_and_conditions_agreed" id="terms_and_conditions_agreed"
           value="1" class="form-check-input" required>
    <label class="form-check-label" for="terms_and_conditions_agreed">
        I have read and agree to the Terms and Conditions
    </label>
</div>
```

**Form Submission**:
```javascript
// terms_and_conditions_text has no `name` attribute and is `disabled`,
// so FormData never includes it — no manual exclusion needed.
// If it were built with `readonly` and a `name` attribute instead, it would
// have to be explicitly deleted before submit, e.g.:
// formData.delete('terms_and_conditions_text');
```

---

## Transaction Device Field - Hardcoded Static Options

**Field ID**: `id="transaction_device"`  
**Name**: `name="transaction_device"`  
**Type**: SELECT dropdown  
**Required**: No  

**Static Options** (slug => name key-value pairs):
```javascript
{
  "Easy-Pay-Direct-Gateway": "Easy-Pay-Direct-Gateway",
  "Another-Payment-Gateway": "Another-Payment-Gateway",
  "Need-terminal/POS-system": "Need-terminal/POS-system",
  "Own-terminal/POS-system": "Own-terminal/POS-system",
  "iPhone/Android": "iPhone/Android",
  "Other": "Other"
}
```

**HTML Example**:
```html
<select name="transaction_device" id="transaction_device" class="form-control custom-select">
  <option value="">Please Select</option>
  <option value="Easy-Pay-Direct-Gateway">Easy-Pay-Direct-Gateway</option>
  <option value="Another-Payment-Gateway">Another-Payment-Gateway</option>
  <option value="Need-terminal/POS-system">Need-terminal/POS-system</option>
  <option value="Own-terminal/POS-system">Own-terminal/POS-system</option>
  <option value="iPhone/Android">iPhone/Android</option>
  <option value="Other">Other</option>
</select>
```

**JavaScript Example** (for population):
```javascript
const transactionDevices = {
  "Easy-Pay-Direct-Gateway": "Easy-Pay-Direct-Gateway",
  "Another-Payment-Gateway": "Another-Payment-Gateway",
  "Need-terminal/POS-system": "Need-terminal/POS-system",
  "Own-terminal/POS-system": "Own-terminal/POS-system",
  "iPhone/Android": "iPhone/Android",
  "Other": "Other"
};

// Build dropdown
Object.entries(transactionDevices).forEach(([slug, name]) => {
  $('#transaction_device').append(`<option value="${slug}">${name}</option>`);
});

// Or pre-populate if value is known
const savedValue = "iPhone/Android";
$('#transaction_device').val(savedValue);
```

---

## Other Interests Capital - Grouped Checkboxes

**Field**: `other_interests_capital`  
**Type**: Checkbox array (multiple selection)  
**Required**: No  
**API Endpoint**: `GET /api/partner/interest-details`

**Response Format** (Grouped by Interest Group):
```json
{
  "success": true,
  "message": "Interest list",
  "data": [
    {
      "group_name": "Capital Investment",
      "interests": [
        { "id": 1, "name": "Equipment Financing" },
        { "id": 2, "name": "Working Capital" },
        { "id": 3, "name": "Growth Capital" }
      ]
    },
    {
      "group_name": "Debt Management",
      "interests": [
        { "id": 4, "name": "Debt Consolidation" },
        { "id": 5, "name": "Refinancing" }
      ]
    }
  ]
}
```

**HTML Implementation** (Grouped Checkboxes):
```html
<div class="interests-section">
  <label>What are your capital interests?</label>
  
  <!-- Dynamically render groups from API response -->
  <div class="interest-group">
    <h4>Capital Investment</h4>
    <div class="form-check">
      <input type="checkbox" name="other_interests_capital" value="1" id="interest_1" class="form-check-input">
      <label class="form-check-label" for="interest_1">Equipment Financing</label>
    </div>
    <div class="form-check">
      <input type="checkbox" name="other_interests_capital" value="2" id="interest_2" class="form-check-input">
      <label class="form-check-label" for="interest_2">Working Capital</label>
    </div>
    <!-- More interests... -->
  </div>
  
  <div class="interest-group">
    <h4>Debt Management</h4>
    <div class="form-check">
      <input type="checkbox" name="other_interests_capital" value="4" id="interest_4" class="form-check-input">
      <label class="form-check-label" for="interest_4">Debt Consolidation</label>
    </div>
    <!-- More interests... -->
  </div>
</div>
```

**JavaScript Implementation** (Render from API):
```javascript
// Fetch interests from API
$.ajax({
    url: '/api/partner/interest-details',
    type: 'GET',
    headers: {
        'Content-Type': 'application/json'
    },
    success: function(response) {
        const container = $('.interests-section');
        
        response.data.forEach((group, index) => {
            // Create group heading
            const groupDiv = $(`<div class="interest-group"><h4>${group.group_name}</h4></div>`);
            
            // Add checkboxes for each interest
            group.interests.forEach((interest) => {
                const checkboxId = `interest_${interest.id}`;
                const checkboxHtml = `
                    <div class="form-check">
                        <input type="checkbox" name="other_interests_capital" 
                               value="${interest.id}" id="${checkboxId}" 
                               class="form-check-input">
                        <label class="form-check-label" for="${checkboxId}">
                            ${interest.name}
                        </label>
                    </div>
                `;
                groupDiv.append(checkboxHtml);
            });
            
            container.append(groupDiv);
        });
    }
});

// On form submission, collect selected interests
const selectedInterests = $('input[name="other_interests_capital"]:checked')
    .map(function() { return $(this).val(); })
    .get();
// selectedInterests = [1, 4, 5, ...] (array of interest IDs)
```

**Form Submission**:
- Submit as array: `other_interests_capital[]` with multiple checkbox values
- Example: `other_interests_capital=1&other_interests_capital=2&other_interests_capital=4`
- Backend receives as array of interest IDs

---

## Dependent Fields

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`).

- **howdidyouhear_other**: Show if howdidyouhear="Other" or "Friend"
- **bad_experience_happened**: Show if bad_experience=1

---

## Form Submission & Final Redirect

**On Success (HTTP 200/201)** - FINAL STEP:
```javascript
// Response example:
{
  "status": true,
  "message": "Signup completed successfully",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "step_count": 6,
  "redirect": "/signup/success"
}

// Redirect to success page
const redirectUrl = response.redirect || '/signup/success';
window.location.href = redirectUrl;
```

**On Validation Error (HTTP 422)**: standard shape — see skill.md → Error Handling. Stay on Step 6, display field errors, do not change URL.

---

## API Integration

**Dropdown Data**:
```
GET /api/partner/referral-sources
GET /api/partner/interest-details
```

**Form Submission**:
```
POST /api/v1/application/step
Headers: None required (see skill.md → Authentication)
Payload:
  step_count: 6
  section: interest_details
  all Step 6 fields
  uuid: {uuid}
Response: { redirect: "/signup/success" }
```

---

## Field Summary

**Total Fields**: 11  
**Required Fields**: 4  
**Conditional Fields**: 2
