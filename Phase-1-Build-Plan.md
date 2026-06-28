# Phase 1: App Validation Specification

**Status:** Complete  
**Spec version shipped:** 1.1.0  
**Completed:** 2026-06-27

This document records the Phase 1 plan and what was delivered. The **normative contract** lives in [APP_PACKAGE_SPEC.md](APP_PACKAGE_SPEC.md) and [schemas/app.schema.json](schemas/app.schema.json). If this plan and those files disagree, trust the spec and schema.

---

## Overview

Phase 1 established `app-validation-spec` as the official documentation-and-contract foundation for an automated app validation system.

The repo defines how every app idea should be packaged so automation tools (n8n, Cursor, Codex, Vercel, Meta Ads, analytics dashboards, landing page templates) can consume the same standardized input.

Phase 1 is **documentation and contract only**. No runtime tooling was built.

---

## Repo structure (shipped)

```txt
app-validation-spec/
├── README.md
├── APP_PACKAGE_SPEC.md
├── CHANGELOG.md
├── Phase-1-Build-Plan.md          # this file
├── schemas/
│   └── app.schema.json
├── templates/
│   ├── app.json
│   └── copy/
│       ├── hero.md
│       ├── features.md
│       └── faq.md
├── examples/
│   ├── minimal-app/               # Focus Timer — draft, inline copy
│   └── full-app/                  # Habit Stack — ready, all sections
└── docs/
    ├── design-philosophy.md
    ├── workflow.md
    ├── n8n-integration-notes.md
    ├── naming-conventions.md
    └── versioning.md
```

---

## Implementation status

| Step | Scope | Status |
|------|-------|--------|
| **A — Contract core** | `README.md`, `APP_PACKAGE_SPEC.md`, `schemas/app.schema.json`, `templates/app.json`, `CHANGELOG.md` | Done |
| **B — Templates & examples** | `templates/copy/`, `examples/minimal-app/`, `examples/full-app/` | Done |
| **C — Supporting docs** | All files in `docs/` | Done |

---

## Shipped spec version: 1.1.0

See [CHANGELOG.md](CHANGELOG.md) for the full 1.1.0 release notes.

### Added in 1.1.0 (backward compatible)

- `landingPage.sections`: `id: "screenshots"` with `source: "media"`
- `mockup.installCommand`, `mockup.devCommand`
- `deployment.vercelDeploymentUrl`, `deployment.lastDeployedAt`
- `appStore.supportUrl`, `appStore.privacyPolicyUrl`
- `mediaAsset.title`, `mediaAsset.description` (screenshot captions)
- `ads.platforms[]`
- Structured `ads.utmTemplate` object (legacy string form still valid)

Packages at `specVersion: "1.0.0"` remain valid if they omit 1.1.0 optional fields.

---

## Key design decisions (implemented)

### Required root fields (JSON Schema)

```
specVersion, appId, status, identity, audience, commerce, landingPage
```

`experiment` is **not** root-required. It is optional in the schema so minimal/draft packages stay valid. It **must be fully populated before `status: ready`** — enforced in Phase 2 validator, not JSON Schema alone.

### experiment model (repo version, not original plan draft)

The shipped model uses automation-friendly structures:

```json
"experiment": {
  "experimentName": "Habit Stack — Q2 2026 Validation",
  "hypothesis": "We believe [audience] will [action] because [reason].",
  "successCriteria": [
    "Cost per email signup below $5",
    "Buy Now click rate above 2%",
    "At least 200 unique landing page visitors"
  ],
  "testBudget": {
    "currency": "USD",
    "amount": 750,
    "durationDays": 14
  },
  "decisionRules": {
    "winnerThreshold": "CPA below $8 AND trial start rate above 15% after min sample size",
    "killThreshold": "CPA above $20 after 50% of budget spent with fewer than 10 signups",
    "minSampleSize": 200,
    "notes": "Pause for manual review if inconclusive at day 10."
  }
}
```

Not shipped: prose-only `decisionRules` (`winner` / `kill` / `iterate`) or numeric-object `successCriteria` from early plan drafts.

### Landing page copy sources

| `source` | Use |
|----------|-----|
| `inline` | Short copy in `app.json` — minimal packages |
| `file` | Markdown in `copy/*.md` — full packages |
| `media` | Screenshot carousel from `media.screenshots` |

Example screenshots section:

```json
{ "id": "screenshots", "enabled": true, "source": "media" }
```

### Mockup (React/Vite JSX)

Paths are relative to the **package root**:

```json
"mockup": {
  "type": "interactive",
  "sourcePath": "mockup/",
  "framework": "react-vite",
  "entryPoint": "mockup/src/App.jsx",
  "installCommand": "npm install",
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "deployCommand": "npx vercel deploy --prod",
  "previewUrl": null
}
```

`devCommand` is for local preview. `installCommand`, `buildCommand`, and `deployCommand` are for automation (Phase 2+).

### Branding

Shipped model uses `branding.theme` (colors, mode, font) and `branding.style` (tone enum + `styleNotes`).

Not shipped: preset theme enums (`liquid-glass`, `apple-light`, etc.) on `theme.style`. Visual presets belong in `styleNotes` or future minor versions if needed.

### ads.utmTemplate

Prefer structured object:

```json
"utmTemplate": {
  "source": "facebook",
  "medium": "paid_social",
  "campaign": "{{appId}}-validation"
}
```

Legacy query string form remains valid for backward compatibility.

---

## app.json top-level structure (1.1.0)

```json
{
  "specVersion": "1.1.0",
  "appId": "your-app-id",
  "status": "draft",
  "identity": {},
  "audience": {},
  "commerce": {},
  "landingPage": {},
  "branding": {},
  "mockup": {},
  "media": {},
  "ads": {},
  "tracking": {},
  "analytics": {},
  "experiment": {},
  "deployment": {},
  "appStore": {}
}
```

Only the first seven sections (through `landingPage`) are required by JSON Schema.

---

## status lifecycle

| Value | Meaning |
|-------|---------|
| `draft` | Work in progress |
| `ready` | Complete; awaiting validation kickoff |
| `validating` | Active experiment |
| `paused` | Temporarily stopped |
| `winner` | Met success criteria |
| `killed` | Failed kill criteria |
| `built` | Shipped to production |

New packages start as `draft`. Automation sets `validating`, writes `deployment` URLs, and may set `winner` or `killed`.

---

## Validation profiles

### Minimal (`examples/minimal-app/`)

- `app.json` only — Focus Timer, `status: draft`, `specVersion: 1.1.0`
- Inline `hero` and `cta` sections
- No `experiment`, `media`, `mockup`, or optional sections

### Full (`examples/full-app/`)

- Habit Stack — all sections, `status: ready`
- File-based copy, `screenshots` via `source: "media"`, structured UTM, full `experiment`
- Placeholder READMEs in `media/` and `mockup/` (no binaries or mockup code in repo)

---

## Canonical documents

| Document | Purpose |
|----------|---------|
| [APP_PACKAGE_SPEC.md](APP_PACKAGE_SPEC.md) | Normative field reference |
| [schemas/app.schema.json](schemas/app.schema.json) | Machine-readable JSON Schema |
| [README.md](README.md) | Repo overview and quick start |
| [CHANGELOG.md](CHANGELOG.md) | Version history |
| [docs/design-philosophy.md](docs/design-philosophy.md) | Design decisions and tradeoffs |
| [docs/workflow.md](docs/workflow.md) | Pipeline stages → spec fields |
| [docs/n8n-integration-notes.md](docs/n8n-integration-notes.md) | Drive layout, parsing, write-back |
| [docs/naming-conventions.md](docs/naming-conventions.md) | `appId`, paths, filenames |
| [docs/versioning.md](docs/versioning.md) | Semver policy and 1.0→1.1 migration |

---

## Known gaps (deferred to Phase 2)

These are documented but **not enforced by JSON Schema** in Phase 1:

- `experiment` required when `status` is `ready` or `validating`
- At least `hero` + `cta` enabled in `landingPage`
- `source: "media"` must pair with `id: "screenshots"` and non-empty `media.screenshots`
- Referenced files and media paths must exist on Drive

Phase 2 validator CLI will enforce these rules.

---

## Out of scope (Phase 2+)

Not built in Phase 1:

- Validator CLI
- n8n workflow JSON
- Landing page generator
- Vercel deployment scripts
- Facebook ad publisher
- Analytics dashboard
- Real mockup deployment
- Real image assets in examples

`deployCommand` values in `app.json` are **config metadata** for future automation, not runnable deploy logic in this repo.

---

## Future pipeline (context)

The App Package is the source of truth. A future n8n pipeline will:

```txt
Read App Package (Google Drive)
  → Validate app.json and required files
  → Deploy mockup app
  → Generate landing page
  → Deploy landing page to Vercel
  → Create Facebook ad assets
  → Track emails and Buy Now clicks
  → Feed metrics into analytics dashboard
  → Decide: build, kill, pause, or continue validating
```

See [docs/workflow.md](docs/workflow.md) for stage-by-stage field mapping.
