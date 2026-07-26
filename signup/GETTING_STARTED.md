# Getting Started - Signup Form Skill

**Quick start guide for using this skill to generate complete signup forms.**

---

## 📋 For Developers: Where to Start

### Day 1: Understanding the Requirements

1. **Read** `skill.md` Overview section (10 min)
   - Understand form structure: 6 steps, 78+ fields
   - Review form overview table

2. **Read** `skill.md` Step 1-2 sections (20 min)
   - Understand each field, validation, conditional logic
   - Note Google Maps autocomplete integration for addresses

3. **Reference** `api-examples.md` (15 min)
   - Understand API endpoints, headers, payloads
   - Note: Two different authentication headers!

### Day 2: Implementation Details

1. **Read** `skill.md` Step 3-6 sections (30 min)
   - Understand each step's fields and validation
   - Note conditional field logic

2. **Read** `DROPDOWNS_REFERENCE.md` (15 min)
   - See all dropdown values (static and API-based)
   - Understand component mapping for autocomplete

3. **Review** `DEPENDENT_FIELDS.md` (20 min)
   - Understand all conditional field relationships
   - Learn nested dependency logic (Level 1 and 2)

### Day 3: Building Forms

1. **Copy** code templates from `reference/api-examples.md`
   - JavaScript fetch examples for all 6 dropdown APIs
   - Complete form submission examples (Steps 1-6)
   - cURL examples for testing
   - Error handling patterns

2. **Reference** `skill.md` for each step
   - Exact field names, validation rules
   - Conditional display logic
   - Submission endpoints
   - Google Maps autocomplete implementation

3. **Follow** testing checklist in `skill.md`
   - Verify each step works
   - Check conditionals trigger correctly
   - Validate API payloads
   - Test Google Maps autocomplete

### Day 4+: Production Deployment

1. **Verify** all functionality:
   - All 6 dropdown APIs returning data
   - All 6 steps submitting correctly
   - All conditional fields working
   - Google Maps autocomplete functioning
   - Error handling for all status codes

2. **Test** edge cases:
   - Different countries with different required fields
   - Multiple revenue model selections
   - Owner 2 visibility when < 51%
   - Both address autocompletes

3. **Deploy** with confidence:
   - All integrations tested
   - Mobile responsive verified
   - Performance validated

---

## 📱 For Product Managers: What You Need to Know

### Features
- **6-step progressive disclosure form** - User fills one step at a time
- **Google Maps address autocomplete** - Legal + physical address auto-population
- **Multi-library integration** - Phone formatting, sliders, date pickers
- **Conditional fields** - Show/hide and nested dependencies
- **Revenue model with nested conditionals** - 2-level dependent hierarchy
- **Two-phase API** - Different headers for different calls
- **Data persistence** - localStorage between page refreshes
- **Country-specific fields** - Dynamic field visibility based on country

### Current Status
- ✅ 78+ fields fully documented
- ✅ 9 integrations documented
- ✅ Complete Google Maps autocomplete implementation
- ✅ All API endpoints documented
- ✅ Complete testing checklist ready
- ✅ Production-ready specification

### Key Metrics
- **Field Count**: 78+ across 6 steps
- **Validation Rules**: 30+ business logic rules
- **API Calls**: 3 form submission + 6 dropdown fetch = 9 total
- **Dependent Fields**: 25+ with multi-level conditionals
- **Google Maps Autocomplete**: 2 instances (legal + physical addresses)

---

## 🔧 For QA: Testing Framework

### Test Order

**Phase 1: Individual Step Testing**
```
✓ Step 1: 9 fields - Account info
✓ Step 2: 28 fields - Company info with autocomplete
✓ Step 3: 13 fields - Product info with slider
✓ Step 4: 32+ fields - Owner info with Owner 2 conditional
✓ Step 5: 10 fields - Banking info
✓ Step 6: 11 fields - Final details
```

**Phase 2: Autocomplete Testing** (NEW in Step 2)
```
✓ Legal address autocomplete shows initially
✓ Selecting address auto-populates all 6 fields
✓ Autocomplete hides after selection
✓ Address fields become required
✓ Physical address autocomplete shows when radio="yes"
✓ Selecting physical address auto-populates 6 fields
✓ Country selectpicker refreshes correctly
```

**Phase 3: Conditional Field Testing** (Use DEPENDENT_FIELDS.md)
```
✓ State dropdown (Step 1) - appears when country=US
✓ Revenue model (Step 2) - checkbox array with nesting
✓ Subscription frequency (Step 2 Level 2) - nested conditional
✓ Physical address section (Step 2) - shown when radio="yes"
✓ Fulfillment vendor (Step 3)
✓ Shopping cart other (Step 3)
✓ Owner 2 section (Step 4) - appears when ownership < 51%
✓ Country-specific fields (Step 5)
✓ Conditional text fields (Step 6)
```

**Phase 4: API Integration Testing** (Use api-examples.md)
```
✓ Dropdown data loads (6 GET calls)
✓ Form submission succeeds (3 POST calls)
✓ UUID persists correctly
✓ Error handling works (422, 401, 403, 500)
✓ Validation messages display
✓ Different headers used (Authorization vs X-API-Key)
```

**Phase 5: End-to-End Testing**
```
✓ Complete form with all fields
✓ Submit all 6 steps
✓ Verify database records created
✓ Check email sent
✓ Confirm redirect to dashboard
✓ Test with different countries
```

### Critical Tests

**MUST PASS**:
1. [ ] Legal address autocomplete works (Step 2)
2. [ ] Physical address autocomplete works (Step 2)
3. [ ] State appears when US selected (Step 1)
4. [ ] Revenue model nested conditional works (Step 2)
5. [ ] Owner 2 appears when < 51% (Step 4)
6. [ ] Slider snaps to 5% increments (Step 3)
7. [ ] Form completes all 6 steps

---

## 🚀 Quick Reference: Most Important Things

### The Two Headers (DON'T GET WRONG!)

```javascript
// FOR FORM SUBMISSION (POST)
headers: {
    'X-API-Key': user.security_key  // ← X-API-Key
}

// FOR DROPDOWN DATA (GET)
headers: {
    'Authorization': user.security_key  // ← Authorization (different!)
}
```

### The Country/State Logic

```javascript
// Country IDs:
1 = United States (show state dropdown)
2 = Canada (hide state dropdown)
3+ = Other countries (hide state dropdown)

// When to show state:
if (country_id === "1") {
    show(".usa_state");  // Show state selector
}
```

### The Physical Address Logic

```
Radio value "no" (default) = Physical address is SAME as legal
  → Physical address section HIDDEN
  → Do NOT submit physical address data
  → Submit: is_physical_address_same_as_legal_address: 1

Radio value "yes" = Physical address is DIFFERENT from legal
  → Physical address section SHOWN
  → Show Google autocomplete for physical address
  → On autocomplete selection, auto-populate 6 fields
  → DO submit physical address data
  → Submit: is_physical_address_same_as_legal_address: 0
```

### The Google Maps Autocomplete Logic

```
LEGAL ADDRESS:
1. Autocomplete textbox shown initially (if no data exists)
2. User searches and selects address from dropdown
3. All 6 address fields auto-populate from Google place object
4. Autocomplete hides, address fields become visible + required

PHYSICAL ADDRESS:
1. Only shown if radio="yes" (different address)
2. Autocomplete textbox shown initially (if no data exists)
3. User searches and selects address from dropdown
4. All 6 physical address fields auto-populate from Google place object
5. Autocomplete hides, address fields become visible + required
```

### The Slider Rules

```javascript
// Slider MUST:
- Only allow values: 0, 5, 10, 15... 100
- Sum all three to exactly 100%
- Have step: 5 configured
- Real-time percentage display

// Three fields:
card_swiped (point of sale)
customer_entered (online)
staff_entered (manually)
```

### The Owner 2 Conditional

```javascript
// Show Owner 2 if:
ownership_percentage_of_owner_1 < 51

// Hide Owner 2 if:
ownership_percentage_of_owner_1 >= 51

// When visible:
All fields required
Must fill name, email, phone, etc.
```

---

## 📚 File Guide: Where to Find What

| What I Need | Where to Look |
|---|---|
| **Full specification** | `skill.md` (production spec with all 78+ fields) |
| **Dependent fields** | `reference/DEPENDENT_FIELDS.md` (all 25+ conditionals) |
| **Step 4 details** | `reference/STEP4_OWNER_INFORMATION.md` (Owner 1 & 2) |
| **API & code** | `reference/api-examples.md` (JavaScript + cURL) |
| **Dropdown values** | `reference/DROPDOWNS_REFERENCE.md` (70+ options + autocomplete) |
| **Google Maps** | `reference/DROPDOWNS_REFERENCE.md` → Google Maps section |
| **Quick start** | This file (GETTING_STARTED.md) |
| **Overview index** | `INDEX.md` (guide to all skill files) |

---

## 🎯 Success Criteria

### For Developers (Before Deployment)
- [ ] All 78+ fields implemented correctly
- [ ] All 6 dropdown APIs returning data
- [ ] Both address autocompletes functioning
- [ ] All 3 form submission endpoints working
- [ ] Error handling for all status codes
- [ ] All conditional fields working (Level 1 & 2)
- [ ] Mobile responsive design verified

### For QA (Before Launch)
- [ ] All individual step tests pass
- [ ] All autocomplete tests pass
- [ ] All conditional field tests pass
- [ ] All API integration tests pass
- [ ] End-to-end flow completes successfully
- [ ] Database records created correctly
- [ ] Cross-browser testing complete
- [ ] Performance testing (< 2s form load, < 1s step submit)

### For Product (Before Live)
- [ ] Form completion rate > 80%
- [ ] Error rate < 2%
- [ ] Average time to complete < 10 minutes
- [ ] User feedback positive
- [ ] No critical bugs reported

---

## 💡 Pro Tips

1. **Use cURL for quick API testing**
   ```bash
   curl -X GET "https://emap.epd.dev/api/partner/countries" \
     -H "Authorization: YOUR_API_KEY"
   ```

2. **Check browser console for errors**
   - Look for dependent-fields.js errors
   - Check Google Maps API errors
   - Monitor network tab for failed requests

3. **Test on real data**
   - Don't just hardcode test values
   - Use actual API responses
   - Test with edge cases (long names, special characters)

4. **Verify localStorage**
   - UUID should persist between page refreshes
   - Check DevTools → Application → localStorage
   - Clear cache if form state broken

5. **Use the testing checklist**
   - Mark off tests as you go
   - Don't skip "optional" features
   - Test both happy path and error cases

---

## 🆘 Troubleshooting Quick Links

| Problem | Where to Look |
|---|---|
| State dropdown not appearing | skill.md Step 1 → Dependent Fields section |
| Legal address not auto-populating | DROPDOWNS_REFERENCE.md → "Legal Address Autocomplete" |
| Physical address autocomplete not showing | skill.md Step 2 → Physical Address section |
| Slider values wrong | skill.md Step 3 → Transaction Entry Methods |
| Owner 2 not showing | STEP4_OWNER_INFORMATION.md → Owner 2 Conditional |
| API returns 403 | api-examples.md → "CRITICAL: Header Differences" |
| API returns 401 | api-examples.md → Check Authorization header |
| Form data lost on refresh | skill.md Step 1 → API Integration |
| Conditional fields not showing | DEPENDENT_FIELDS.md → all conditional logic |
| Google Maps autocomplete not working | DROPDOWNS_REFERENCE.md → Google Maps section |

---

## 📞 Support Resources

**For exact field specifications**: Read `skill.md` Step 1-6 sections

**For API details**: See `reference/api-examples.md` or `reference/DROPDOWNS_REFERENCE.md`

**For dropdown values**: Check `reference/DROPDOWNS_REFERENCE.md`

**For conditional logic**: Review `reference/DEPENDENT_FIELDS.md`

**For Google Maps autocomplete**: See `reference/DROPDOWNS_REFERENCE.md` → Google Maps section

**For Step 4 Owner details**: Check `reference/STEP4_OWNER_INFORMATION.md`

**For testing**: Follow checklist in `skill.md` Testing Checklist section

---

## 🎓 Learning Path (Recommended Reading Order)

### Beginner (Non-technical)
1. This file (GETTING_STARTED.md) - 10 min
2. skill.md → Form Overview table - 5 min
3. INDEX.md → File guide - 5 min

### Developer
1. This file (GETTING_STARTED.md) - 15 min
2. skill.md → Complete read - 60 min
3. reference/DEPENDENT_FIELDS.md → All conditionals - 20 min
4. reference/api-examples.md → Code examples - 20 min
5. reference/DROPDOWNS_REFERENCE.md → Implementation - 20 min
6. reference/STEP4_OWNER_INFORMATION.md → Step 4 details - 20 min

### QA/Tester
1. This file (GETTING_STARTED.md) - 15 min
2. skill.md → Field tables for all steps - 30 min
3. reference/DEPENDENT_FIELDS.md → Conditional testing - 20 min
4. reference/api-examples.md → API error codes - 10 min
5. reference/DROPDOWNS_REFERENCE.md → Test data - 15 min

### Project Manager
1. This file (GETTING_STARTED.md) - 15 min
2. skill.md → Form Overview table - 5 min
3. skill.md → Feature highlights (Google Maps, conditionals) - 10 min

---

**Total Documentation**: ~8000 lines  
**Total Fields**: 78+  
**Total Endpoints**: 9 (6 dropdown fetch + 3 form submission)
**Total Integrations**: 10 (Google Maps, intl-tel-input, ion-rangeslider, date picker, etc.)
**Production-Ready**: ✅ Yes  
**Status**: ✅ Documentation Complete, ✅ Ready for Implementation

---

**Start here, then pick the path that matches your role above.**

Good luck! 🚀
