# EasyPayDirect Signup Skills

This skill is for **partners** who want to earn commission by referring merchants to EasyPayDirect. Give it to your AI agent and it builds a branded referral signup page that you can host anywhere (Vercel, Netlify, your own site, etc.). Every merchant who signs up through your page is attributed to you via your partner key, so the signups you drive earn you commission.

## Installation

**What you'll need**

- **A partner API key (optional)**, from the [EasyPayDirect partner portal](https://emap.easypaydirect.com/signup/partner). It's what attributes signups to you for commission.
- **An AI coding assistant** (Cursor, Claude Code, etc.). This is what actually builds the page from the skill. Or app builders like Replit, v0, etc. work too, as long as you can give them the `SKILL.md` contents to build from.
- **Node.js**, only for the `npx` install method below; not needed if you copy the skill manually. Get it at [nodejs.org](https://nodejs.org).

> Not comfortable with a terminal? You can skip the commands entirely. Open your AI assistant, give it the skill (paste the link to `SKILL.md`), and ask it to build the page for you.

**1. Get the skill**

*Option A: `npx skills` (recommended)*

> **Before you run this, you need Node.js installed.** It's what provides the `npx` command. Download it from [nodejs.org](https://nodejs.org) (pick the "LTS" version and click through the installer), then reopen your terminal. To check it worked, run `node --version`; if it prints a version number you're set. If `npx` still isn't found after installing, close and reopen the terminal.

```bash
npx skills add arpit-dapton/easypaydirect-signup-skill-v1
```

*Option B: download it (no terminal needed)*

1. Click the green **Code** button → **Download ZIP**, then unzip it.
2. The skill is the `skills/signup` folder inside. Point your agent at it (next step), or drop that folder wherever your agent reads skills from.

**2. Point your agent at it**

Tell your AI assistant to build the form from `SKILL.md`. A few examples:

*Claude:*
```
Build the signup form using this specification: skills/signup/SKILL.md
```

*Any other agent:*
```
Generate the signup form described in skills/signup/SKILL.md.
```

`SKILL.md` is the single entry point. It links out to every other file the agent needs, so that one path is all you have to give it.

## Answer the Gate Questions

Before it writes any code, the skill stops and asks:

1. *Do you have a partner API key?* Determines whether `partner_key` gets sent in the Step 1 payload.
2. *How do you want the sign-up process to work for merchants on your site?* Asked in plain language, no internal jargon, with exactly three options:

   1. **All in one place**: the merchant fills out their entire application on your website, from start to finish. They never have to leave your site.
   2. **Quick start, then handed off**: the merchant only enters their basic contact info on your website. As soon as they submit that, they're automatically taken to EasyPayDirect's own website to finish the rest of their application there.
   3. **Quick start, then we email you a link**: the merchant enters their basic contact info on your website, and instead of being redirected, they get an email with a link to continue on EasyPayDirect whenever they're ready.

## Folder Structure

```
easypaydirect-signup-skill-v1/
├── README.md ...................................... This file
│
└── skills/
    └── signup/ ..................................... The skill
        ├── SKILL.md ................................ ⭐ Entry point: navigation hub & spec
        │
        ├── steps/ .................................. Step 1 field + conditional docs
        │
        └── reference/ .............................. Supporting docs
            ├── api-examples.md
            ├── DROPDOWNS_REFERENCE.md
            └── BACKEND_DEPENDENCIES.md
```

This is the flat layout the `npx skills` CLI expects (`skills/<name>/SKILL.md`), so any skill placed under `skills/` here is installable with `npx skills add` out of the box.

## Current Skills

| Skill | Entry point |
|-------|-------------|
| EasyPayDirect Merchant Signup | [`skills/signup/SKILL.md`](skills/signup/SKILL.md) |

---

**Last updated**: 2026-09-04
