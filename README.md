# EMAP Signup Skills

EMAP signup: a reusable skill and specification for AI code generation.

## Installation

**1. Get the skill**

*Option A: `npx skills` (recommended)*

```bash
npx skills add arpit-dapton/emap-signup-skills
```

This is the [open agent skills CLI](https://skills.sh): it detects which coding agents you have installed (Cursor, Codex, Junie, and 70+ others), lets you pick which to install to, and symlinks or copies `skills/signup/` into the right place for each. Use `--list` first to preview what it finds without installing anything:

```bash
npx skills add arpit-dapton/emap-signup-skills --list
```

*Option B: copy it manually*

```bash
git clone https://github.com/arpit-dapton/emap-signup-skills.git emap-signup-skills
cp -r emap-signup-skills/skills/signup <your-agent's-skills-dir>/signup
```

(Where that is depends on your agent: see the CLI's [Supported Agents](https://github.com/vercel-labs/skills#supported-agents) table for the exact path. Any folder your agent can read works, even outside that convention.)

**2. Point your agent at it**

```
Generate the signup form using this specification:
path/to/signup/SKILL.md
```

`SKILL.md` is the single entry point. It links out to every step file and reference doc it needs, so the agent never has to guess where the rest of the spec lives.

## Answer the Gate Questions

Before it writes any code, the skill stops and asks:

1. *Do you have a partner API key?* Determines whether `partner_key` gets sent in the Step 1 payload.
2. *How do you want the sign-up process to work for merchants on your site?* Asked in plain language, no internal jargon, with exactly three options:

   1. **All in one place**: the merchant fills out their entire application on your website, from start to finish. They never have to leave your site. *(Internally: the regular flow, all 6 steps, completed on this form.)*
   2. **Quick start, then handed off**: the merchant only enters their basic contact info on your website. As soon as they submit that, they're automatically taken to EMAP's own website to finish the rest of their application there. *(Internally: only Step 1 gets built on your site; Steps 2–6 are skipped entirely, and the redirect lands on EMAP's existing resume page.)*
   3. **All in one place, with a "save and finish later" option**: same as option 1 (the whole application happens on your website), but if a merchant gets interrupted partway through, they can request an email with a link that lets them pick up right where they left off. *(Internally: all 6 steps, same as option 1, plus a "finish later" action that emails a resume link, available from Step 2 onward, once there's an application to resume.)*

These aren't optional: they're one-way doors (a signup submitted without `partner_key` can never be attributed to a partner after the fact, and the flow choice determines which pages get built at all), so the skill is written to refuse to proceed until a human actually answers. See the gate question and its full implementation table in [`skills/signup/SKILL.md`](skills/signup/SKILL.md).

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

## Current Skills

| Skill | Entry point |
|-------|-------------|
| EMAP Merchant Signup | [`skills/signup/SKILL.md`](skills/signup/SKILL.md) |

---

**Last updated**: 2026-09-02
