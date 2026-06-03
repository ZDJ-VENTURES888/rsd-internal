# RSD Internal — Campaign & Client Pages

**Private repository · ZDJ-VENTURES888**  
Real Space Digital · Zack Jackson & Joel Kaminski

---

## What This Repo Is

A private GitHub Pages site for internal RSD campaign assets, client clipboards, and production tools. Only Zack and Joel have access. Files are never indexed or linked externally.

**Live URL:** https://zdj-ventures888.github.io/rsd-internal/

---

## Repo Structure

```
rsd-internal/
├── index.html                          ← Main dashboard (the homepage)
├── higgs-field/
│   └── Higgs_Field_Campaign_Clipboard.html
├── viral-prompts/
│   └── Viral_Prompt_Library_Clipboard.html
├── berriz/                             ← Drop Berriz files here
├── [next-client]/                      ← Create folders per client as needed
├── README.md                           ← This file
└── HANDOFF.md                          ← Deployment instructions for Claude Desktop
```

---

## How to Add a New Client or Page

1. Create a folder: `[client-name]/` at the repo root
2. Drop the HTML clipboard file inside it
3. Add a new card to `index.html` following the existing card pattern
4. Commit and push — GitHub Pages publishes automatically within ~60 seconds

---

## Current Pages

| Page | Path | Status |
|------|------|--------|
| Dashboard | `index.html` | Live |
| Higgs Field Campaign Clipboard | `higgs-field/Higgs_Field_Campaign_Clipboard.html` | Active Campaign |
| Viral Prompt Library | `viral-prompts/Viral_Prompt_Library_Clipboard.html` | Active Tool |
| Berriz Design Build | `berriz/` | Awaiting files |

---

## Brand System

- **Navy:** `#0B2545`
- **Gold:** `#C9A961`
- **Cream:** `#F0EDE6`
- **Display font:** Cormorant Garamond
- **Body font:** Calibri / Outfit

---

## Internal Use Only

These files are not for client distribution. All pages are marked with internal badges. Do not share direct GitHub Pages URLs with external parties.

Base prompt credit for Viral Prompt Library: [harry0703/MoneyPrinterTurbo](https://github.com/harry0703/MoneyPrinterTurbo)
