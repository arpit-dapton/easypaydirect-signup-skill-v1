# EMAP Signup Skills

A production-ready Claude Code skill for generating EMAP's 6-step merchant signup form — not a toy prompt, a full specification an agent can build against.

Most "build me a signup form" prompts produce something that looks right and breaks on the first edge case: a conditional field that should only show up for Canadian owners, a re-submission that silently duplicates a record, an error response the agent never accounted for. This skill exists because that gap is exactly where merchant-onboarding forms go wrong.

Instead of describing the form once and hoping the agent infers the rest, this repo documents the form the way you'd hand it to a new engineer: every field, every conditional, every API response shape, verified against the actual backend rather than assumed. Hack on it, trim it, or fork it for your own multi-step form — it's built to be read and edited, not just executed.

## Installation

**1. Get the skill**

*Option A — `npx skills` (recommended)*

```bash
npx skills add arpit-dapton/emap-signup-skills
```

This is the [open agent skills CLI](https://skills.sh) — it detects which coding agents you have installed (Claude Code, Cursor, Codex, and 70+ others), lets you pick which to install to, and symlinks or copies `skills/signup/` into the right place for each (e.g. `.claude/skills/signup/` for Claude Code). Use `--list` first to preview what it finds without installing anything:

```bash
npx skills add arpit-dapton/emap-signup-skills --list
```

*Option B — copy it manually*

```bash
git clone https://github.com/arpit-dapton/emap-signup-skills.git emap-signup-skills
cp -r emap-signup-skills/skills/signup .claude/skills/signup
```

(Any path works — `.claude/skills/` is just where Claude Code looks for skills automatically. If you're using a different agent, any folder it can read works too.)

**2. Point your agent at it**

```
Generate the signup form using this specification:
.claude/skills/signup/SKILL.md
```

`SKILL.md` is the single entry point. It links out to every step file and reference doc it needs — the agent never has to guess where the rest of the spec lives.

**3. Answer the two gate questions**

Before it writes any code, the skill stops and asks:

1. *Do you have a partner API key?* — determines whether `partner_key` gets sent in the Step 1 payload.
2. *How do you want the sign-up process to work for merchants on your site?* — asked in plain language, no internal jargon, with exactly three options:

   1. **All in one place** — the merchant fills out their entire application on your website, from start to finish. They never have to leave your site. *(Internally: the regular flow — all 6 steps, completed on this form.)*
   2. **Quick start, then handed off** — the merchant only enters their basic contact info on your website. As soon as they submit that, they're automatically taken to EMAP's own website to finish the rest of their application there. *(Internally: only Step 1 gets built on your site — Steps 2–6 are skipped entirely, and the redirect lands on EMAP's existing resume page.)*
   3. **All in one place, with a "save and finish later" option** — same as option 1 (the whole application happens on your website), but if a merchant gets interrupted partway through, they can request an email with a link that lets them pick up right where they left off. *(Internally: all 6 steps, same as option 1, plus a "finish later" action that emails a resume link — available from Step 2 onward, once there's an application to resume.)*

These aren't optional — they're one-way doors (a signup submitted without `partner_key` can never be attributed to a partner after the fact, and the flow choice determines which pages get built at all), so the skill is written to refuse to proceed until a human actually answers. See the gate question and its full implementation table in [`skills/signup/SKILL.md`](skills/signup/SKILL.md).

## Why This Skill Exists

### #1: The agent guesses instead of asking

**The problem.** Signup forms have decisions only a human can make — do you have a partner key, which of three supported flows do you want — and a "just build something reasonable" agent will pick one silently instead of asking.

**The fix.** `SKILL.md` opens with two hard gates (see Installation above) that override any auto-mode/no-clarifying-questions instinct the agent might have. It waits for a real answer before scaffolding anything.

### #2: The spec drifts from what the backend actually does

**The problem.** Most form specs are written from memory or from a design doc, not from the code that will actually receive the request — so they miss things like "every success response is HTTP 200, never 201" or "this endpoint is 500 on error while every other one is 400 for the same case."

**The fix.** Every status code, error shape, and backend endpoint in this skill (see the [Error Handling](skills/signup/SKILL.md#error-handling) table and [`reference/BACKEND_DEPENDENCIES.md`](skills/signup/reference/BACKEND_DEPENDENCIES.md)) is checked against the actual controller code it describes, not inferred from convention. Where something is unverified, the doc says so instead of quietly asserting it.

### #3: Conditional logic lives far from the field it affects

**The problem.** A dependent-field reference file that lives apart from the step it belongs to means an agent building Step 4 has to hold Step 4's fields *and* a separate conditionals doc in its head at once — the classic recipe for "changed the field, forgot the condition."

**The fix.** Conditional/dependent-field logic is written inline in each step's own file, right next to the field it controls (Step 2 is the one exception, split into a companion `_CONDITIONALS.md` because its nesting runs three levels deep). One file, one source of truth, per step.

### #4: Nobody checks the seams between steps

**The problem.** Each step's fields can be correct in isolation while the flow still breaks — a stale `uuid` after refresh, a duplicate submission on back-navigation, a persisted `country` value that Steps 2–5 quietly assume is still there.

**The fix.** `SKILL.md`'s [Testing Checklist](skills/signup/SKILL.md#testing-checklist) calls out exactly these cross-step seams — refresh behavior, re-submission, and the specific conditional combinations that are easy to get right per-field and wrong end-to-end.

## Reference

### The skill

- **[`skills/signup/SKILL.md`](skills/signup/SKILL.md)** — Navigation hub. Form overview, the two mandatory gate questions, auth, error handling, cross-step guidance (step progression, refresh behavior, re-submission, form-data persistence), and the testing checklist. Everything else is linked directly from here.

### Steps

One file per step; conditional logic lives inline in each rather than in a separate reference.

| Step | File | Fields | Covers |
|------|------|--------|--------|
| 1 | [`STEP1_ACCOUNT_INFORMATION.md`](skills/signup/steps/STEP1_ACCOUNT_INFORMATION.md) | 11 | Contact + company basics, phone formatting, optional partner attribution |
| 2 | [`STEP2_COMPANY_INFORMATION.md`](skills/signup/steps/STEP2_COMPANY_INFORMATION.md) / [`_CONDITIONALS.md`](skills/signup/steps/STEP2_COMPANY_INFORMATION_CONDITIONALS.md) | 26 | Legal + physical address, revenue model with nested conditionals |
| 3 | [`STEP3_PRODUCT_INFORMATION.md`](skills/signup/steps/STEP3_PRODUCT_INFORMATION.md) | 13 | Fulfillment details, transaction-split slider (multiples of 5, sums to 100) |
| 4 | [`STEP4_OWNER_INFORMATION.md`](skills/signup/steps/STEP4_OWNER_INFORMATION.md) / [`_FIELDS.md`](skills/signup/steps/STEP4_OWNER_INFORMATION_FIELDS.md) / [`_IMPLEMENTATION.md`](skills/signup/steps/STEP4_OWNER_INFORMATION_IMPLEMENTATION.md) | 45+ | Primary contact, Owner 1 & conditional Owner 2, driver's license, financial history |
| 5 | [`STEP5_BANKING_INFORMATION.md`](skills/signup/steps/STEP5_BANKING_INFORMATION.md) | 9 | Routing validation, country-specific fields |
| 6 | [`STEP6_FINAL_DETAILS.md`](skills/signup/steps/STEP6_FINAL_DETAILS.md) | 11 | Interest selection, terms & conditions acceptance |

### Supporting reference

- **[`reference/api-examples.md`](skills/signup/reference/api-examples.md)** — Copy-paste JavaScript & cURL for every dropdown and submission endpoint.
- **[`reference/DROPDOWNS_REFERENCE.md`](skills/signup/reference/DROPDOWNS_REFERENCE.md)** — API-backed dropdown values (countries, states, industry types, referral sources, shopping carts, interests).
- **[`reference/DROPDOWNS_STATIC_REFERENCE.md`](skills/signup/reference/DROPDOWNS_STATIC_REFERENCE.md)** — Static dropdown values and a plain-dropdown implementation path.
- **[`reference/BACKEND_DEPENDENCIES.md`](skills/signup/reference/BACKEND_DEPENDENCIES.md)** — What has to actually be deployed server-side before Flow Options 2 and 3 work.
- **[`reference/TERMS_AND_CONDITIONS_TEXT.md`](skills/signup/reference/TERMS_AND_CONDITIONS_TEXT.md)** — Point-in-time snapshot of the T&C text shown on Step 6.

## Folder Structure

```
emap-signup-skills/
├── README.md ...................................... This file
│
└── skills/
    └── signup/ ..................................... The skill
        ├── SKILL.md ................................ ⭐ Entry point: navigation hub & spec
        │
        ├── steps/ .................................. Per-step field + conditional docs (Steps 1–6)
        │
        └── reference/ .............................. Supporting docs
            ├── api-examples.md
            ├── DROPDOWNS_REFERENCE.md
            ├── DROPDOWNS_STATIC_REFERENCE.md
            ├── BACKEND_DEPENDENCIES.md
            └── TERMS_AND_CONDITIONS_TEXT.md
```

This is the flat layout the `npx skills` CLI expects (`skills/<name>/SKILL.md`), so any skill placed under `skills/` here is installable with `npx skills add` out of the box.

## Adding a New Skill

This repo currently holds one skill, but it's structured so another can sit alongside it:

```
emap-signup-skills/
└── skills/
    └── {skill-name}/
        ├── SKILL.md ................ Entry point & spec
        └── reference/
            └── {topic}.md
```

Then add it to the Reference section above.

## Current Skills

| Skill | Status | Entry point |
|-------|--------|-------------|
| EMAP Merchant Signup | ✅ Production ready | [`skills/signup/SKILL.md`](skills/signup/SKILL.md) |

---

**Last updated**: 2026-09-02
