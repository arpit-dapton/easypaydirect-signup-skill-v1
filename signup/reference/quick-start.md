# Signup Form Creation Skill - Quick Start

## What's Included

A comprehensive, production-ready skill document for creating the merchant signup form with all 6 steps, dynamic field handling, API integration, and industry-standard best practices.

**Main Skill Files**:
- `.claude/skills/signup-form-creation.md` - Master skill document
- `.claude/skills/signup-fields-reference.json` - Machine-readable specification
- `.claude/API_INTEGRATION_GUIDE.md` - Complete API endpoint reference
- `.claude/SIGNUP_SKILL_README.md` - This quick reference guide

## Key Features

✅ **API-Driven Dropdowns**
- All dropdown values fetched from `/api/partner/` endpoints
- 6 API endpoints for countries, states, industries, referral sources, devices, interests
- All responses use `slug` (value) and `name` (label) format
- Complete API integration guide included

✅ **Complete Form Coverage**
- All 6 steps with detailed field specifications
- 59+ form fields across all steps
- 10+ dynamic field behaviors

✅ **Dynamic Field Management**
- Country-based field visibility (Formation State, Business Register Number)
- Industry-based conditional fields (Industry Other)
- Revenue model triggers (Subscription Frequency)
- Owner count dependencies (second owner appears when first < 50%)
- Referral source dependencies (Additional info for Other/Friend/Event)
- Provider experience dependencies (Provider name field)
- Canada-specific fields (Institution Number, Currency)

✅ **Industry Standards**
- Progressive disclosure (show only relevant fields)
- Context-aware validation
- Auto-population support from external sources
- Pre-filled auth fields
- Accessibility considerations
- Responsive design patterns

✅ **Optimized Context Window**
- Organized by step with clear hierarchy
- Reusable pattern documentation
- Partial includes referenced (not duplicated)
- Smart field reference table
- Data model examples

## Excluded Features (Per Your Requirements)

❌ **Plaid Persona Connection** - Step 4 identity verification
❌ **Dynamic Processor Fields** - Conditional complex forms (Step 6 modal)

## How to Use This Skill

### For Creating the Signup Form UI

When you give this skill to an AI to create the form:

```
"Create the signup form using the signup-form-creation skill. 
Generate HTML form structure with proper field types, validations, 
and dynamic behaviors. Use Bootstrap for responsive layout. 
Focus on the form structure only - no API integration yet."
```

### AI Will Understand

1. **Field Structure**: Exact field names, types, validation rules, placeholders
2. **Dynamic Behaviors**: When to show/hide fields based on user selections
3. **Validation Rules**: Required fields, format validation, custom rules
4. **User Experience**: Progressive disclosure, helpful hints, error messages
5. **Data Flow**: Form submission endpoints, field mapping, pre-population support

### Key Dynamic Patterns to Understand

```
// Formation Country triggers Formation State visibility
Formation Country = "1" (US) → Show Formation State field

// Industry selection triggers Industry Other field
Industry Type = "Other" → Show Industry Type Other field

// Revenue Model checkbox triggers Subscription Frequency
Subscription selected → Show Subscription Frequency dropdown
Subscription Frequency = "3" (Other) → Show Subscription Frequency Other

// Ownership percentage affects second owner visibility
Owner 1 Ownership < 50% → Show Owner 2 section + hint

// Referral source affects detail field
"How did you find us" IN [Other, Friend, Live Event] → Show Additional Information

// Bad experience affects provider field
Bad experience = "Yes" → Show provider name field

// Country affects banking fields
Country = "Canada" → Show Institution Number & Currency
```

## Step-by-Step Summary

### Step 1: Account Information (7 fields)
- Owner details (first name, last name, email, phone)
- Company name, website
- **Formation country** (API: `/api/partner/countries`)
- **Formation state** (API: `/api/partner/states`, conditional on country=US)
- Conditional annual sales (variant-specific)
- Referral code (optional)

### Step 2: Company Information (15 fields)
- Legal name, DBA name
- **Industry type** (API: `/api/partner/industry-types`) with conditional "Other" field
- Conditional annual sales
- Customer service phone
- Company location type, formation date
- Legal structure
- Federal Tax ID with dynamic label
- Conditional business register number
- Revenue model checkboxes
- Subscription frequency with conditional "Other"
- Full company address
- Full physical address

### Step 3: Product Information (5 fields)
- **3-way slider**: Card swiped % | Customer entered % | Staff entered %
- Fulfillment methods
- Average transaction amount
- Highest transaction amount
- Products/services description
- Highest transaction description

### Step 4: Owner Information (22 fields)
- Primary contact details (partial)
- Owner #1 (required):
  - First/Last name, email (pre-filled from auth)
  - Title, SSN, phone
  - Ownership %, DOB
  - Financial history, home address, license details
- Owner #2 (conditional, shows if owner 1 < 50%):
  - Same structure as Owner #1
  - Optional except when triggered by ownership rules

### Step 5: Banking Information (2 fields + partials)
- Conditional Institution Number (Canada only)
- Conditional Currency radio buttons (Canada only)
- Transaction information (partial)

### Step 6: Final Details (8 fields)
- **How did you find us** (API: `/api/partner/referral-sources`) + conditional additional info
- Multiple merchant accounts needed (required)
- **Transaction device preference** (API: `/api/partner/shopping-carts`, optional)
- Bad experience with provider (required + conditional provider name)
- **Interests/capital preferences** (API: `/api/partner/interest-details`, checkboxes)
- Terms and conditions agreement (required)
- Optional: Dynamic fields modal (processor-specific)

## Field Types Used

- **Text Input**: Names, companies, descriptions
- **Email**: Contact emails
- **Tel/Phone**: Phone numbers with intl-tel-input library
- **Number**: Percentages, amounts
- **Date**: DOB, formation date (YYYY-MM-DD format)
- **URL**: Website (with validation)
- **Currency**: Amounts (Cleave.js formatting with commas)
- **Dropdown Select**: Countries, states, industries, devices
- **Searchable Select**: Industry, countries (live search)
- **Checkbox**: Revenue models, interests
- **Radio Button**: Currency choice (USD vs CAD)
- **Range Slider**: 3-way transaction distribution
- **Textarea**: Descriptions, comments

## Validation Patterns

```javascript
// Standard required validation
field: { required: true }

// Email validation
field: { required: true, validate_email: true }

// URL validation
field: { required: true, validUrl: true }

// Phone validation (intl-tel-input)
field: { required: true, validatePhone: true }

// Currency with custom max
field: { required: true, ValidateAnnualSales: true }

// Percentage distribution (sum to 100%)
field: { validPercentage: true, maxPercentage: true }

// Text length constraints
field: { required: true, minlength: 5, maxlength: 60 }

// Numeric bounds
field: { required: true, min: 1, max: 100 }
```

## API Integration

### Dropdown Data Sources
All dropdown values are fetched from REST APIs:

**Base URL**: `/api/partner`

**Endpoints**:
- `GET /api/partner/countries` - Formation Country (Step 1) & Company Country (Step 2)
- `GET /api/partner/states` - Formation State (Step 1, conditional on US)
- `GET /api/partner/industry-types` - Industry Type (Step 2)
- `GET /api/partner/referral-sources` - How did you find us (Step 6)
- `GET /api/partner/shopping-carts` - Transaction Device (Step 6)
- `GET /api/partner/interest-details` - Help Your Company Grow (Step 6)

**Response Format**:
```json
{
  "data": [
    { "slug": "value_code", "name": "Display Label" }
  ]
}
```

**Implementation**:
- Use `slug` as form field value
- Use `name` as dropdown display label
- See `API_INTEGRATION_GUIDE.md` for complete implementation details

### Auto-Save Behavior

**Step 1 Only** (when from_lander = true):
- Triggers when all required fields are filled
- On blur from last field
- Debounced to prevent rapid requests
- Endpoint: POST /signup/auto-save

## Pre-Population Support

Many fields can be pre-populated from:
- User authentication data (first name, last name, email)
- External APIs (company info, address via Google Maps)
- Cookies/session (referral codes, partner info)
- Previous form saves

Check `isAuto()` and `autoSourceText()` helper patterns in skill.

## Context Window Tips

This skill is optimized for context efficiency:

1. **Organized hierarchy**: Steps → Cards → Fields
2. **No duplication**: Partials referenced with descriptions
3. **Reusable patterns**: Common validation/formatting rules
4. **Data model example**: JSON structure for field definitions
5. **Reference tables**: Step-by-step field count and complexity
6. **Testing checklist**: All dynamic scenarios listed

**Total Skill Size**: ~24KB
**Estimated Token Usage**: ~6,000 tokens
**Optimized For**: Single passing to AI to generate complete form

## Example Usage in Prompt

```
"I need you to create the signup form HTML using the signup-form-creation skill 
from .claude/skills/signup-form-creation.md

Requirements:
- Use Bootstrap 4 for responsive grid
- Implement all 59 form fields across 6 steps
- Add all dynamic field behaviors (state visibility, conditional fields, etc.)
- Use proper form validation messages
- Include auto-save on step 1 if from_lander
- Support pre-populated fields with source indicators
- Ensure accessibility (labels, hints, ARIA)
- Focus on UI/UX only - no API integration

Start with Step 1, then move to Step 2, Step 3, etc.
Ensure each step has proper form structure and field dependencies."
```

## What's NOT Included

- API endpoints/integration (will be added later)
- Form submission logic (will be added later)
- Server-side validation (will be added later)
- Backend database schema (will be added later)
- JavaScript event handlers (will be added later)
- CSS styling beyond Bootstrap (will be added later)
- Authentication/authorization logic (will be added later)

## Next Steps

1. ✅ **Skill created** - Complete form specification
2. ⏳ **Share with AI** - Give this skill to generate form UI
3. ⏳ **Form HTML** - Generate responsive form structure
4. ⏳ **Validation JS** - Add form validation and dynamic behavior
5. ⏳ **API Integration** - Add form submission endpoints
6. ⏳ **Testing** - Verify all dynamic fields work correctly

---

**Skill Version**: 1.0
**Created**: 2025-07-15
**Last Updated**: 2025-07-15
**Optimized For**: AI form creation without context limitations
