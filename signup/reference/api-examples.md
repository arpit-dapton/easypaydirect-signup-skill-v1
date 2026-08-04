# API Implementation Examples

Ready-to-use code examples for all API calls in the signup form.

---

## 🔑 CRITICAL: Header Differences

**This is the #1 mistake that causes 403/401 errors!**

| Endpoint Type | Header | Value | Error if Wrong |
|---|---|---|---|
| **Dropdown Data** | `Authorization` | user.security_key | 403 (invalid key) |
| **Form Submission** | `X-API-Key` | user.security_key | 401 (key required) |

Both use the SAME value (security_key) but DIFFERENT header names!

---

## Dropdown APIs

### Load Countries (Example 1)

**JavaScript**:
```javascript
const apiKey = sessionStorage.getItem('api_key');

async function loadCountries() {
  try {
    const response = await fetch('https://emap.epd.dev/api/partner/countries', {
      method: 'GET',
      headers: {
        'Authorization': apiKey,  // ← Authorization header (NOT X-API-Key!)
        'Content-Type': 'application/json'
      }
    });

    if (response.status === 403) {
      console.error('Invalid API key');
      return [];
    }

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
  -H "Authorization: YOUR_API_KEY_HERE" \
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
  const apiKey = sessionStorage.getItem('api_key');
  
  const response = await fetch('https://emap.epd.dev/api/partner/industry-types', {
    headers: { 'Authorization': apiKey }
  });
  
  const result = await response.json();
  return result.data;  // [{name: "Retail", slug: "retail"}, ...]
}
```

### Load States (Step 2)

```javascript
async function loadStates() {
  const apiKey = sessionStorage.getItem('api_key');
  
  const response = await fetch('https://emap.epd.dev/api/partner/states', {
    headers: { 'Authorization': apiKey }
  });
  
  const result = await response.json();
  return result.data;  // [{name: "California", code: "CA"}, ...]
}
```

### Load Other Dropdowns

Same pattern for all dropdown endpoints:

```javascript
// Referral Sources
fetch('https://emap.epd.dev/api/partner/referral-sources', {
  headers: { 'Authorization': apiKey }
})

// Shopping Carts / Transaction Devices
fetch('https://emap.epd.dev/api/partner/shopping-carts', {
  headers: { 'Authorization': apiKey }
})

// Interest Details
fetch('https://emap.epd.dev/api/partner/interest-details', {
  headers: { 'Authorization': apiKey }
})
```

---

## Form Submission APIs

### Step 1: Submit Account Information

**JavaScript**:
```javascript
const apiKey = sessionStorage.getItem('api_key');

async function submitStep1(formData) {
  try {
    // Step 1 payload - ONLY these 9 fields
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

    // Submit to API
    const response = await fetch('https://emap.epd.dev/api/v1/signup', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-API-Key': apiKey,  // ← X-API-Key header (NOT Authorization!)
        'Accept': 'application/json'
      },
      body: JSON.stringify(payload)
    });

    if (response.status === 201) {
      // ✅ Success
      const result = await response.json();
      console.log('Account created!');
      console.log('UUID:', result.uuid);
      
      // Store UUID for future steps
      localStorage.setItem('signup_uuid', result.uuid);
      
      // Navigate to Step 2
      window.location.href = `/step/2/${result.uuid}`;
      
      return result;
      
    } else if (response.status === 401 || response.status === 403) {
      // ❌ Authentication error
      console.error('Invalid API key. Please log in again.');
      return null;
      
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
      // ❌ Other error
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
  -H "X-API-Key: YOUR_API_KEY_HERE" \
  -d '{
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com",
    "phone": "+1-555-1234",
    "name": "Acme Corp",
    "website": "https://acme.com",
    "country": "US",
    "annual_sales": 500000,
    "business_state": "CA"
  }'

# Success Response (201):
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
async function submitStep2(step2Data) {
  const apiKey = sessionStorage.getItem('api_key');
  const uuid = localStorage.getItem('signup_uuid');
  
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

  const response = await fetch('https://emap.epd.dev/api/v1/application/step', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-API-Key': apiKey
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

All steps use the same endpoint and headers:

```javascript
async function submitStep(stepNumber, sectionName, stepData) {
  const apiKey = sessionStorage.getItem('api_key');
  const uuid = localStorage.getItem('signup_uuid');
  
  const payload = {
    step_count: stepNumber,
    uuid: uuid,
    section: sectionName,  // "product_info", "ownership_info", "banking_info", "interest_details"
    ...stepData
  };

  const response = await fetch('https://emap.epd.dev/api/v1/application/step', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-API-Key': apiKey
    },
    body: JSON.stringify(payload)
  });

  if (response.ok) {
    const result = await response.json();
    if (result.next_step_url) {
      window.location.href = result.next_step_url;
    } else if (result.redirect) {
      window.location.href = result.redirect;  // Step 6 redirects to dashboard
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
});
```

---

## Complete Flow Example

```javascript
// 1. After login, store API key
sessionStorage.setItem('api_key', user.security_key);

// 2. Step 1 - Load dropdown
const countries = await loadCountries();  // Uses Authorization header

// 3. Step 1 - Submit form
const step1Result = await submitStep1(formData);  // Uses X-API-Key header
// Result: returns UUID, stored in localStorage

// 4. Step 2 - Load dropdowns
const industries = await loadIndustries();  // Uses Authorization header

// 5. Step 2 - Submit form
const step2Result = await submitStep(2, 'company_info', formData);  // Uses X-API-Key

// ... repeat for Steps 3-6

// Final: Step 6 redirects to dashboard
```

---

## Error Handling Quick Reference

| Status | Meaning | Action |
|--------|---------|--------|
| 200/201 | ✅ Success | Proceed to next step |
| 400 | Bad request | Check payload format |
| 401 | ❌ Auth error | Check X-API-Key header for form submission |
| 403 | ❌ Auth error | Check Authorization header for dropdowns |
| 422 | Validation error | Display field errors to user |
| 500 | Server error | Retry or report |

---

## Quick Copy-Paste Template

```javascript
// Dropdown fetch
const response = await fetch('https://emap.epd.dev/api/partner/{ENDPOINT}', {
  headers: { 'Authorization': sessionStorage.getItem('api_key') }
});

// Form submission
const response = await fetch('https://emap.epd.dev/api/v1/{ENDPOINT}', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-API-Key': sessionStorage.getItem('api_key')
  },
  body: JSON.stringify(payload)
});
```

Ready to copy-paste! 🎉
