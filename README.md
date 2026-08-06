# Claude Code Skills

Professional, reusable skills and specifications for AI code generation.

---

## 📋 Signup Form Skill

**Location**: `./signup/`

A comprehensive skill for creating a merchant signup form with 6 steps, 115+ fields, dynamic behaviors, and complete API integration.

### ⭐ Main Skill File

- **`skill.md`** - Navigation hub: form overview, required implementation parameters, authentication, error handling, cross-step guidance, and testing checklist. Links directly to every other file below.

### 📚 Per-Step Documentation

`steps/` — one file per step (dependent-field logic lives inline in each step file, not in a separate reference):

- `STEP1_ACCOUNT_INFORMATION.md`
- `STEP2_COMPANY_INFORMATION.md` / `STEP2_COMPANY_INFORMATION_CONDITIONALS.md`
- `STEP3_PRODUCT_INFORMATION.md`
- `STEP4_OWNER_INFORMATION.md` / `_FIELDS.md` / `_IMPLEMENTATION.md`
- `STEP5_BANKING_INFORMATION.md`
- `STEP6_FINAL_DETAILS.md`

### 📚 Reference Documentation

`reference/`:

- `DROPDOWNS_REFERENCE.md` - API-backed dropdown values (countries, states, industry types, referral sources, shopping carts, interests)
- `DROPDOWNS_STATIC_REFERENCE.md` - Static dropdown values, plain-dropdown implementation, conversion reference, testing checklist
- `api-examples.md` - Copy-paste JavaScript & cURL code

---

## 🚀 How to Use

### For AI Code Generation (Recommended)

Share the main skill file with AI — it links to everything else needed:

```
"Generate the signup form using this specification:
.claude/skills/signup/skill.md"
```

### For Developer Reference

1. Read `skill.md` for the overview, auth, error handling, and cross-step guidance
2. Follow its links to the specific step file(s) you're implementing
3. Use `reference/api-examples.md` for copy-paste request code
4. Use `reference/DROPDOWNS_REFERENCE.md` / `DROPDOWNS_STATIC_REFERENCE.md` for dropdown values

---

## 📁 Folder Structure

```
skills/
├── README.md ................................. This file (index)
│
└── signup/ ................................... Signup form skill
    ├── skill.md ............................. ⭐ MAIN: Navigation hub & specification
    │
    ├── steps/ ............................... Per-step field documentation (Steps 1-6)
    │
    └── reference/ .......................... Supporting documentation
        ├── DROPDOWNS_REFERENCE.md
        ├── DROPDOWNS_STATIC_REFERENCE.md
        └── api-examples.md
```

**Key**: `skill.md` is the single navigation hub — every other file is linked directly from it, one level deep. There is no separate machine-readable spec or dependent-fields reference; those were removed as duplicates of what's already in the step files (see `skill.md`'s git history if you need the old JSON spec).

---

## ✨ Signup Skill Features

✅ **Complete Coverage**: 6 steps, 115+ fields  
✅ **Dynamic Fields**: Conditional show/hide behaviors documented per step  
✅ **API Integration**: 6 dropdown endpoints + 3 form-submission endpoints documented  
✅ **Validation**: All rules specified  
✅ **No Auth Required**: Only Step 1 accepts an optional, non-blocking partner-attribution field

**API Endpoints**:
- `GET /api/partner/countries`
- `GET /api/partner/states`
- `GET /api/partner/industry-types`
- `GET /api/partner/referral-sources`
- `GET /api/partner/shopping-carts`
- `GET /api/partner/interest-details`

---

## 🎯 Creating New Skills

To add a new skill, follow this structure:

```
skills/
└── {skill-name}/
    ├── skill.md ............................ Main specification & navigation hub
    └── reference/
        ├── {feature}-guide.md
        └── notes.md
```

Then update this README with the new skill.

---

## 📊 Current Skills

| Skill | Status | Files |
|-------|--------|-------|
| Signup Form | ✅ Ready | skill.md |

---

## 📞 Support

**Main specification**: `./signup/skill.md`  
**API docs**: `./signup/reference/api-examples.md`

---

**Status**: ✅ Production Ready  
**Last Updated**: 2026-08-05
