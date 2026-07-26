# Signup Form Skill - Quick Reference

Production-ready specification for the complete 6-step merchant signup form with 116+ fields, Google Maps address autocomplete, and multi-level conditionals.

---

## 📚 Documentation Files

### Main Specification
- **skill.md** - Complete form specification (116+ fields across 6 steps, validations, APIs)
- **reference/STEP4_OWNER_INFORMATION.md** - Detailed Step 4 specification (45 owner fields, primary contact, 2 Google autocompletes)

### Implementation Guides
- **reference/DEPENDENT_FIELDS.md** - All conditional/dependent field logic with code examples
- **reference/DROPDOWNS_REFERENCE.md** - All dropdown values (API + static) 
- **reference/api-examples.md** - Copy-paste JavaScript & cURL code
- **reference/ENDPOINTS.md** - API endpoints, headers, payloads, error codes

---

## 🚀 Quick Start

### For Developers
1. Read `skill.md` - Understand all fields and requirements
2. Read `reference/DEPENDENT_FIELDS.md` - Implement conditional logic
3. Copy code from `reference/api-examples.md` - Use working implementations
4. Reference `reference/ENDPOINTS.md` - Verify API calls

### For QA/Testing
1. Read `skill.md` → Testing Checklist section
2. Reference `reference/DEPENDENT_FIELDS.md` - Test all conditionals
3. Use `reference/ENDPOINTS.md` - Verify error handling

---

## 📋 What's Included

✅ **All 6 Steps** - Account → Company → Product → Owners → Banking → Final

✅ **116+ Fields** - Complete field specifications with validation rules

✅ **Google Maps Autocomplete** - 4 instances (Step 2: legal + physical, Step 4: Owner 1 + Owner 2)

✅ **Multi-Level Conditionals** - 40+ conditional show/hide rules with implementation code

✅ **API Integration** - 9 endpoints (3 form submission + 6 dropdown fetch)

✅ **Dropdown Values** - 8 API-based + 12 static dropdown options

✅ **Code Examples** - JavaScript & cURL ready to use

✅ **Error Handling** - All HTTP status codes documented

---

## 📊 Form Structure

| Step | Fields | Key Features |
|------|--------|--------------|
| 1 | 9 | Phone formatting (intl-tel-input), country/state selection |
| 2 | 28 | Google Maps autocomplete (legal + physical), revenue model with 2-level conditionals |
| 3 | 13 | Fulfillment details, transaction slider (step: 5) |
| 4 | 45+ | Primary contact, Owner 1, conditional Owner 2, Google autocomplete (2), driver's license, bankruptcy |
| 5 | 10 | Routing validation, country-specific fields |
| 6 | 11 | Interest selection, terms acceptance |
| 7 | - | Display only (no submission) |
| **Total** | **116+** | |

---

## 🔑 Critical Information

**Two Different Authentication Headers** (Don't mix them up!):
```
Form Submission (POST):  X-API-Key: {user.security_key}
Dropdown Data (GET):     Authorization: {user.security_key}
```

**Country ID for Conditionals** (Not country code!):
```
USA = "1"
Canada = "2"
Mexico = "3"
```

**Required Libraries**:
- intl-tel-input (phones)
- Google Maps Places (addresses)
- ion-rangeslider (slider with step: 5)
- dependent-fields.js (conditionals)
- Date picker library

---

**Status**: Production Ready ✅  
**Last Updated**: 2026-07-22  
**Total Documentation**: 7 files, ~8000 lines
**Field Coverage**: 100% (116+ fields from actual MOS implementation)  
**Autocomplete Instances**: 4 (all with component mapping documented)
**Multi-Level Conditionals**: 5+ (Revenue Model Level 2, Bankruptcy Level 2 per owner, Owner 2 visibility)
