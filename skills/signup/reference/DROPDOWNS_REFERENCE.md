# API Dropdowns Reference

API-backed dropdown values used throughout the signup form. See [DROPDOWNS_STATIC_REFERENCE.md](DROPDOWNS_STATIC_REFERENCE.md) for static (non-API) dropdown values, dropdown UI implementation, and conversion/testing reference.

⚠️ **None of the endpoints below require an auth header** (see skill.md → Authentication).

## Contents
- API-Based Dropdowns (Countries, States, Industry Types, Referral Sources, Shopping Carts, Interest Details)
- Value vs Label

---

## API-Based Dropdowns

### Countries (Step 1, Step 2, Step 4, Step 5)
**Endpoint**: `GET /api/partner/countries`

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
- Show driver_license_state and driver_license_expiration_date (Step 4 owner) if country = "US" — `license` itself is always visible/required regardless of country

---

### US States (Step 1, Step 5)
**Endpoint**: `GET /api/partner/states`
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

**Continue to**: [DROPDOWNS_STATIC_REFERENCE.md](DROPDOWNS_STATIC_REFERENCE.md) for static dropdown values, dropdown UI implementation, conversion reference, and testing checklist.
