# API Implementation Examples

Ready-to-use code examples for all API calls in the signup form.

---

## Authentication

⚠️ **None of these endpoints require authentication.** Do not send `X-API-Key` or `Authorization` headers to any dropdown or form-submission endpoint.

The one exception: Step 1 accepts an **optional `partner_key` field in the payload body** (not a header) for non-blocking partner attribution. Ask the implementer whether they have a partner API key before including it — see [skill.md](../skill.md) → "STOP — Ask Before Writing Any Code".

---

## Dropdown APIs

### Load Countries (Example 1)

**JavaScript**:
```javascript
async function loadCountries() {
  try {
    const response = await fetch('https://emap.epd.dev/api/partner/countries', {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json'
      }
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const result = await response.json();
    return result.data;  // [{name: "US", code: "US"}, ...]
  } catch (error) {
    console.error('Failed to load countries:', error);
    return [];
  }
}

// Populate dropdown
const countries = await loadCountries();
const countrySelect = document.querySelector('select[name="country"]');
countries.forEach(country => {
  const option = document.createElement('option');
  option.value = country.code;
  option.textContent = country.name;
  countrySelect.appendChild(option);
});
```

**cURL**:
```bash
curl -X GET "https://emap.epd.dev/api/partner/countries" \
  -H "Content-Type: application/json"

# Response:
# {
#   "success": true,
#   "message": "Country list",
#   "data": [
#     { "name": "United States", "code": "US" },
#     { "name": "Canada", "code": "CA" }
#   ]
# }
```

### Load Industry Types

```javascript
async function loadIndustries() {
  const response = await fetch('https://emap.epd.dev/api/partner/industry-types');
  const result = await response.json();
  return result.data;  // [{name: "Retail", slug: "retail"}, ...]
}
```

### Load States (Step 2)

```javascript
async function loadStates() {
  const response = await fetch('https://emap.epd.dev/api/partner/states');
  const result = await response.json();
  return result.data;  // [{name: "California", code: "CA"}, ...]
}
```

### Load Other Dropdowns

Same pattern for all dropdown endpoints — no headers required:

```javascript
// Referral Sources
fetch('https://emap.epd.dev/api/partner/referral-sources')

// Shopping Carts / Transaction Devices
fetch('https://emap.epd.dev/api/partner/shopping-carts')

// Interest Details
fetch('https://emap.epd.dev/api/partner/interest-details')
```

---

## Form Submission APIs

### Step 1: Submit Account Information

**JavaScript**:
```javascript
async function submitStep1(formData) {
  try {
    const payload = {
      first_name: formData.firstName,
      last_name: formData.lastName,
      email: formData.email,
      phone: formData.phone,
      name: formData.companyName,
      website: formData.website,
      country: formData.country,
      annual_sales: parseInt(formData.annualSales),
      business_state: formData.businessState || null
    };

    // OPTIONAL: only include partner_key if the implementer has a partner API key
    // (ask them first — see skill.md → "STOP — Ask Before Writing Any Code"). Omit the
    // field entirely if they don't have one; do not send an empty string.
    if (formData.partnerKey) {
      payload.partner_key = formData.partnerKey;
    }

    // Submit to API (no auth header required)
    const response = await fetch('https://emap.epd.dev/api/v1/signup', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json'
      },
      body: JSON.stringify(payload)
    });

    if (response.status === 200) {
      // ✅ Success
      const result = await response.json();
      console.log('Account created!');
      console.log('UUID:', result.uuid);

      // Persist the uuid for future steps (storage mechanism is your choice)

      // Navigate to Step 2
      window.location.href = `/step/2/${result.uuid}`;

      return result;

    } else if (response.status === 422) {
      // ❌ Validation errors
      const error = await response.json();
      console.error('Validation errors:', error.errors);

      // Display field-level errors
      Object.entries(error.errors).forEach(([field, messages]) => {
        const input = document.querySelector(`[name="${field}"]`);
        if (input) {
          input.classList.add('is-invalid');
          const errorEl = input.parentElement.querySelector('.error-text');
          if (errorEl) {
            errorEl.textContent = messages[0];
          }
        }
      });
      return null;

    } else {
      // ❌ Other error (400 bad request, 403 blocked region, etc.)
      const error = await response.json();
      console.error('Error:', error.message);
      return null;
    }
  } catch (error) {
    console.error('Network error:', error);
    alert('Network error. Please try again.');
    return null;
  }
}

// Usage
document.getElementById('signupForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  
  const formData = {
    firstName: document.querySelector('[name="first_name"]').value,
    lastName: document.querySelector('[name="last_name"]').value,
    email: document.querySelector('[name="email"]').value,
    phone: document.querySelector('[name="phone"]').value,
    companyName: document.querySelector('[name="name"]').value,
    website: document.querySelector('[name="website"]').value,
    country: document.querySelector('[name="country"]').value,
    annualSales: document.querySelector('[name="annual_sales"]').value,
    businessState: document.querySelector('[name="business_state"]')?.value
  };
  
  const result = await submitStep1(formData);
  if (result) {
    console.log('Proceeding to Step 2...');
  }
});
```

**cURL**:
```bash
curl -X POST "https://emap.epd.dev/api/v1/signup" \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com",
    "phone": "+1-555-1234",
    "name": "Acme Corp",
    "website": "https://acme.com",
    "country": "US",
    "annual_sales": 500000,
    "business_state": "CA",
    "partner_key": "OPTIONAL — only if the implementer has one, omit otherwise"
  }'

# Success Response (200):
# {
#   "success": true,
#   "uuid": "550e8400-e29b-41d4-a716-446655440000",
#   "step_count": 1,
#   "message": "Account created successfully"
# }
```

### Step 2: Submit Company Information

**JavaScript**:
```javascript
async function submitStep2(step2Data, uuid) {
  const payload = {
    step_count: 2,
    uuid: uuid,
    section: "company_info",
    legal_name: step2Data.legalName,
    name: step2Data.dbaName,
    industry_type: step2Data.industryType,
    customer_service_telephone_number: step2Data.customerServicePhone,
    business_location: step2Data.businessLocation,
    business_formed: step2Data.businessFormed,
    business_organized: step2Data.businessOrganized,
    federal_tax_id: step2Data.federalTaxId,
    business_register_number: step2Data.businessRegisterNumber || null,
    // ... other Step 2 fields
    is_physical_address_same_as_legal_address: step2Data.sameAddress ? 0 : 1
  };

  // No auth header required
  const response = await fetch('https://emap.epd.dev/api/v1/application/step', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(payload)
  });

  if (response.ok) {
    const result = await response.json();
    window.location.href = result.next_step_url;
    return result;
  } else {
    const error = await response.json();
    console.error('Error:', error.errors || error.message);
    return null;
  }
}
```

### Steps 3-6: Same Pattern

All steps use the same endpoint, and none require an auth header:

```javascript
async function submitStep(stepNumber, sectionName, stepData, uuid) {
  const payload = {
    step_count: stepNumber,
    uuid: uuid,
    section: sectionName,  // "product_info", "ownership_info", "banking_info", "interest_details"
    ...stepData
  };

  const response = await fetch('https://emap.epd.dev/api/v1/application/step', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(payload)
  });

  if (response.ok) {
    const result = await response.json();
    if (result.next_step_url) {
      window.location.href = result.next_step_url;
    } else if (result.redirect) {
      window.location.href = result.redirect;  // Step 6 redirects to the success page
    }
    return result;
  } else {
    const error = await response.json();
    console.error('Error:', error.errors || error.message);
    return null;
  }
}

// Usage
await submitStep(3, 'product_info', {
  card_swiped: 0,
  customer_entered: 100,
  staff_entered: 0,
  average_transaction_amount: 200,
  highest_transaction_amount: 1000,
  describe_services: "...",
  describe_highest_transaction: "..."
}, uuid);
```

---

## Complete Flow Example

```javascript
// 1. Step 1 - Load dropdown (no auth header)
const countries = await loadCountries();

// 2. Step 1 - Submit form (no auth header)
const step1Result = await submitStep1(formData);
// Result: returns uuid — persist it however your implementation prefers

// 3. Step 2 - Load dropdowns (no auth header)
const industries = await loadIndustries();

// 4. Step 2 - Submit form (no auth header)
const step2Result = await submitStep(2, 'company_info', formData, step1Result.uuid);

// ... repeat for Steps 3-6

// Final: Step 6 redirects to the success page
```

---

## Error Handling Quick Reference

| Status | Meaning | Action |
|--------|---------|--------|
| 200 | ✅ Success (every success response uses 200, never 201) | Proceed to next step |
| 400 | Bad request | Check payload format |
| 403 | Blocked region (Step 1 geo-check only) | Show `response.message` to the user |
| 404 | Application/company not found for the `uuid` | `uuid` is stale/invalid — restart from Step 1 |
| 422 | Validation error | Display field errors to user |
| 500 | Server error | Retry or report |

See [skill.md](../skill.md) → Error Handling for the exact response shapes and a worked `handleStepResponse()` example.

---

## Quick Copy-Paste Template

```javascript
// Dropdown fetch (no auth header required)
const response = await fetch('https://emap.epd.dev/api/partner/{ENDPOINT}');

// Form submission (no auth header required)
const response = await fetch('https://emap.epd.dev/api/v1/{ENDPOINT}', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(payload)
});
```

Ready to copy-paste! 🎉
