---
name: signup-form-creation
description: Create a complete 6-step merchant signup form with all fields, validations, and integrations
---

# Merchant Signup Form Specification

Production-ready specification for building the complete 6-step signup form.

---

## Form Overview

| Step | Title | File | Fields | Features |
|------|-------|------|--------|----------|
| 1 | Account Information | [STEP1_ACCOUNT_INFORMATION.md](steps/STEP1_ACCOUNT_INFORMATION.md) | 9 | Phone formatting, country/state selection |
| 2 | Company Information | [STEP2_COMPANY_INFORMATION.md](steps/STEP2_COMPANY_INFORMATION.md) | 26 | Google Maps autocomplete (legal + physical), revenue model with nested conditionals (country/state collected in Step 1) |
| 3 | Product Information | [STEP3_PRODUCT_INFORMATION.md](steps/STEP3_PRODUCT_INFORMATION.md) | 13 | Fulfillment details, transaction slider (multiples of 5) |
| 4 | Owner Information | [STEP4_OWNER_INFORMATION.md](steps/STEP4_OWNER_INFORMATION.md) | 45+ | Primary contact, Owner 1 & conditional Owner 2, Google Maps autocomplete (2), driver's license, financial history |
| 5 | Banking Information | [STEP5_BANKING_INFORMATION.md](steps/STEP5_BANKING_INFORMATION.md) | 10 | Routing validation, country-specific fields |
| 6 | Final Details | [STEP6_FINAL_DETAILS.md](steps/STEP6_FINAL_DETAILS.md) | 11 | Interest selection, terms acceptance |

---

## Quick Reference Files

**Supporting Documentation**:
- [reference/DEPENDENT_FIELDS.md](reference/DEPENDENT_FIELDS.md) - Complete conditional field logic
- [reference/DROPDOWNS_REFERENCE.md](reference/DROPDOWNS_REFERENCE.md) - All dropdown values (API + static)
- [reference/api-examples.md](reference/api-examples.md) - Copy-paste JavaScript & cURL code
- [GETTING_STARTED.md](GETTING_STARTED.md) - Quick start guide for developers
- [INDEX.md](INDEX.md) - File index and navigation

---

## Global Implementation

### Authentication Headers (CRITICAL)

**Form Submission (POST)**:
```javascript
headers: {
    'Content-Type': 'application/json',
    'X-API-Key': user.security_key,
    'Accept': 'application/json'
}
```

**Dropdown Data (GET)**:
```javascript
headers: {
    'Authorization': user.security_key,
    'Content-Type': 'application/json'
}
```

---

### State Management

```javascript
// After Step 1, store UUID
localStorage.setItem('signup_uuid', response.uuid);

// Retrieve for subsequent steps
const uuid = localStorage.getItem('signup_uuid');

// Clear on completion
localStorage.removeItem('signup_uuid');
```

---

### Form Submission Pattern

1. Collect form data from all visible fields
2. Validate client-side
3. Filter: Remove hidden/disabled fields
4. Add step_count, section, uuid
5. POST to API
6. Navigate to next step or dashboard

---

### Required Libraries

```html
<!-- All Steps -->
<script src="/assets/js/dependent-fields.js"></script>

<!-- Step 1 -->
<script src="https://cdn.jsdelivr.net/npm/intl-tel-input@22.0.2/build/js/intl-tel-input.min.js"></script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/intl-tel-input@22.0.2/build/css/intlTelInput.css">

<!-- Step 2, 4 — Google Maps Places (address autocomplete). REQUIRED, load synchronously (no async/defer) BEFORE the autocomplete init script. Use the real key from config, NOT a literal placeholder. -->
<script src="https://maps.google.com/maps/api/js?key={{ config('app.google_map_key') }}&libraries=places" type="text/javascript"></script>

<!-- Step 2, 4 — bootstrap-select: country selectpicker the autocomplete populates via .selectpicker('val'|'refresh') -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-select@1.13.14/dist/css/bootstrap-select.min.css">
<script src="https://cdn.jsdelivr.net/npm/bootstrap-select@1.13.14/dist/js/bootstrap-select.min.js"></script>

<!-- Step 2 — Cleave.js: Federal Tax ID input masking -->
<script defer src="https://cdnjs.cloudflare.com/ajax/libs/cleave.js/1.6.0/cleave.min.js"></script>

<!-- Step 3 -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/ion-rangeslider/2.3.1/ion.rangeSlider.min.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/ion-rangeslider/2.3.1/css/ion.rangeSlider.min.css">

<!-- Date Picker (Steps 2, 4, 5) -->
<!-- Include your date picker library -->

<!-- Form Validation -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.19.5/jquery.validate.min.js"></script>
```

---

### URL Routing & Navigation

**URL Pattern**: `signup/step/{step_number}/{uuid}`

**After Each Step Submission** - Redirect to next step:
```
Step 1 Success → redirect to: signup/step/2/{uuid}
Step 2 Success → redirect to: signup/step/3/{uuid}
Step 3 Success → redirect to: signup/step/4/{uuid}
Step 4 Success → redirect to: signup/step/5/{uuid}
Step 5 Success → redirect to: signup/step/6/{uuid}
Step 6 Success → redirect to: /dashboard/merchant?landing=1
```

**On Page Refresh** - Load current step from URL:
```javascript
// Extract step from URL: /signup/step/3/550e8400-e29b-41d4-a716-446655440000
const urlParams = new URLSearchParams(window.location.pathname);
const currentStep = urlParams.split('/')[3];  // e.g., "3"
const uuid = urlParams.split('/')[4];  // e.g., "550e8400-e29b-41d4-a716-446655440000"

// Load and display the current step
loadStep(currentStep, uuid);

// Store UUID for API calls
localStorage.setItem('signup_uuid', uuid);
```

**Initial URL** (Step 1):
```
GET /signup/step/1/{uuid}
```

**Implementation Notes**:
- UUID stays the same across all 6 steps
- URL changes after SUCCESSFUL submission only
- Failed validation does NOT change URL
- Refresh shows current step (not reset to step 1)
- Last step (Step 6) redirects to dashboard instead of another step

---

### Error Handling

```javascript
// API Response Status Codes
201/200: Success - proceed to next step
400: Bad request - check payload format
401/403: Auth error - check headers
422: Validation error - display field errors (don't change URL)
500: Server error - retry or report
```

---

## Field Summary

**Total Fields**: 116+ (11+26+13+45+10+11 across all 6 steps)  
**Required Fields**: 70+  
**Conditional Fields**: 40+ (multi-level dependencies)  
**Google Maps Autocomplete Instances**: 4  
  - Step 2: Legal address + Physical address
  - Step 4: Owner 1 home address + Owner 2 home address  
**API Dropdowns**: 8  
**Static Dropdowns**: 12  
**Phone Fields**: 4 (intl-tel-input)  
**Date Fields**: 8  
**Radio/Checkbox Fields**: 12+  
**Masked/Encrypted Fields**: SSN (Owner 1 & 2), Federal Tax ID  
**Validations**: 40+ business logic rules  
**Multi-Level Conditionals**: Revenue Model (Step 2), Bankruptcy (Steps 4)

---

## Developer Quick Start

1. **Begin with overview**: This file (skill.md) for form structure
2. **Check step details**: Go to the specific step file in `steps/` folder
3. **Reference conditionals**: See [DEPENDENT_FIELDS.md](reference/DEPENDENT_FIELDS.md)
4. **Copy code examples**: Use [reference/api-examples.md](reference/api-examples.md)
5. **Check dropdown values**: Reference [reference/DROPDOWNS_REFERENCE.md](reference/DROPDOWNS_REFERENCE.md)

---

**Production Ready** ✅  
**Last Updated**: 2026-07-22  
**Documentation Structure**: Main index + 6 step-specific files + 3 reference files  
**Lines per file**: All under 500 lines (follows skill best practices)
