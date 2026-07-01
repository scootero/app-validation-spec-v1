# App Package Specification

**Version:** 1.3.0  
**Status:** Normative  
**Schema:** [schemas/app.schema.json](schemas/app.schema.json)

This document defines the official **App Package** contract for the automated app validation system. Every app idea is represented as a folder (the App Package) with `app.json` as its canonical manifest.

## Goals

The spec must be:

- Easy for a human to fill out
- Easy for AI to generate
- Easy for n8n to parse
- Strict enough to validate later
- Flexible enough for many app categories
- Versioned and backward-compatible
- Reusable for landing pages, ads, analytics, App Store metadata, and dashboards

## App Package folder

An App Package on Google Drive (or any storage) uses the `appId` as the folder name:

```txt
{appId}/
├── app.json              # Required manifest
├── copy/                 # Optional markdown landing copy
│   ├── hero.md           # Headline, subheadline, body
│   ├── benefits.md       # Short value props for hero / benefit grid
│   ├── features.md       # Feature list
│   └── faq.md            # FAQ items
├── media/                # Optional images and video
├── mockup/               # Optional interactive mockup source
└── README.md             # Optional human notes (not validated)
```

Paths in `app.json` are relative to the package root unless noted otherwise.

## app.json overview

`app.json` is a single JSON object. The root has three required scalars and several section objects.

### Root fields

| Field | Required | Description |
|-------|----------|-------------|
| `specVersion` | Yes | Semver of this specification (currently `1.3.0`; see [CHANGELOG.md](CHANGELOG.md)) |
| `appId` | Yes | Stable kebab-case identifier; matches folder name |
| `status` | Yes | Lifecycle state (see [Status lifecycle](#status-lifecycle)) |

### Top-level sections

| Section | Required | Description |
|---------|----------|-------------|
| `identity` | Yes | App name, description, category, platform |
| `audience` | Yes | Target users and pain points |
| `commerce` | Yes | Pricing model and CTA button text |
| `landingPage` | Yes | Landing page structure and copy sources |
| `branding` | No | Theme colors and tone |
| `mockup` | No | Mockup source project and preview URL |
| `media` | No | Icon, screenshots, and other assets |
| `ads` | No | Paid social ad copy variants |
| `tracking` | No | Webhook placeholders and custom events |
| `analytics` | No | Dashboard and funnel identifiers |
| `experiment` | No* | Validation hypothesis, budget, decision rules |
| `deployment` | No | Generated URLs and platform IDs |
| `appStore` | No | Reserved App Store metadata |

\* `experiment` is optional in the JSON Schema but **must be fully populated before setting `status` to `provisioning`**. The Phase 2 validator will enforce this.

---

## Status lifecycle

The `status` field drives pipeline branching in n8n and records where an app idea is in the validation funnel.

| Value | Meaning | Typical next step |
|-------|---------|-------------------|
| `draft` | Work in progress; not ready for automation | Author completes `app.json` and `experiment` |
| `provisioning` | Package complete; awaiting webhook provisioning | n8n provisions `tracking.webhookUrl` |
| `ready` | Webhook provisioned; awaiting validation kickoff | n8n picks up and validates |
| `validating` | Active experiment (ads running, traffic flowing) | Monitor analytics |
| `paused` | Experiment temporarily stopped | Manual review or budget hold |
| `winner` | Met success criteria; proceed to build | Set `status` to `built` after shipping |
| `killed` | Failed kill criteria; archive experiment | No further automation |
| `built` | Winner app shipped to production | Optional `appStore` population |

**Rules:**

- New packages start as `draft`.
- Humans set `provisioning` when the package is complete (experiment, ads, analytics IDs).
- n8n provisions `tracking.webhookUrl` during `provisioning`, then sets `ready`.
- Automation sets `validating`, writes `deployment` URLs, and may set `winner` or `killed` per `experiment.decisionRules`.
- `appId` must never change after the first deploy.

---

## identity

Describes what the app is.

| Field | Required | Type | Notes |
|-------|----------|------|-------|
| `appName` | Yes | string | Display name (1–80 chars) |
| `tagline` | No | string | Short hook for hero and ads (≤120 chars) |
| `badgeText` | No | string | Hero badge label (≤80 chars), e.g. *Coming soon to the App Store* |
| `description` | Yes | string | One-paragraph summary (10–500 chars) |
| `category` | Yes | enum | See [Categories](#categories) |
| `categoryLabel` | If `category` is `other` | string | Free-text category name |
| `platform` | Yes | enum | `ios`, `android`, `web`, `cross-platform` |
| `tags` | No | string[] | Discovery and filtering tags |

### Categories

`productivity`, `health-fitness`, `finance`, `education`, `entertainment`, `social`, `utilities`, `lifestyle`, `business`, `developer-tools`, `other`

---

## audience

Describes who the app is for.

| Field | Required | Type | Notes |
|-------|----------|------|-------|
| `primary` | Yes | string | Primary audience description |
| `landingPhrase` | No | string | Short phrase for landing *Built for …* line (≤120 chars) |
| `secondary` | No | string | Secondary audience |
| `painPoints` | No | string[] | Problems the app solves |
| `personas` | No | object[] | `{ name, summary }` per persona |

---

## commerce

Pricing and call-to-action labels used on landing pages and ads.

### pricing

| Field | Required | Type | Notes |
|-------|----------|------|-------|
| `model` | Yes | enum | `free`, `freemium`, `paid`, `subscription` |
| `currency` | If paid/freemium/subscription | string | ISO 4217 (e.g. `USD`) |
| `amount` | If paid/freemium/subscription | number | Price amount; `0` for free tier |
| `period` | If subscription | enum | `one-time`, `monthly`, `yearly` |
| `trialDays` | No | integer | Free trial length |
| `headlineLabel` | No | string | Pricing label before amount, e.g. *Get for* |

### cta

| Field | Required | Type | Notes |
|-------|----------|------|-------|
| `primaryText` | Yes | string | Main button label |
| `secondaryText` | No | string | Secondary action (e.g. Learn More) |
| `buyNowText` | Yes | string | Purchase-intent button label |
| `waitlistText` | No | string | Waitlist or early-access button label |
| `emailPlaceholder` | No | string | Email input placeholder (prefer `landingPage.sections[cta].inline.placeholder`) |

---

## branding

Visual theme and writing tone for landing pages and mockups.

### theme

| Field | Required | Type | Notes |
|-------|----------|------|-------|
| `mode` | Yes | enum | `light`, `dark`, `auto` |
| `landingStyle` | No | enum | Landing theme preset: `liquid-glass`, `apple-light`, `apple-dark`, `midnight`, `aurora`, `black-titanium`, `minimal-white` |
| `primaryColor` | Yes | string | Hex color (e.g. `#2563EB`) |
| `accentColor` | No | string | Hex color |
| `accentName` | No | enum | Landing accent palette: `violet`, `blue`, `emerald`, `rose`, `amber`, `cyan` |
| `fontFamily` | No | string | CSS font stack |

### style

| Field | Required | Type | Notes |
|-------|----------|------|-------|
| `tone` | Yes | enum | `professional`, `playful`, `minimal`, `bold`, `custom` |
| `styleNotes` | No | string | Free-text guidance for designers and AI |

---

## mockup

Supports both **source project info** (for building and deploying the interactive mockup) and **deployed preview info** (written back by automation).

| Field | Required | Type | Notes |
|-------|----------|------|-------|
| `type` | No | enum | `static`, `prototype`, `interactive` |
| `sourcePath` | No | string | Path to mockup source folder (e.g. `mockup/`) |
| `framework` | No | string | e.g. `react`, `vue`, `html-static` |
| `entryPoint` | No | string | Entry file (e.g. `mockup/src/App.jsx`) |
| `installCommand` | No | string | Dependency install command (e.g. `npm install`) |
| `buildCommand` | No | string | Shell command to build before deploy |
| `devCommand` | No | string | Shell command to run mockup locally |
| `deployCommand` | No | string | Shell command to deploy the mockup |
| `previewUrl` | No | string \| null | Live preview URL; `null` until deployed |
| `baseWidth` | No | integer | Mockup iframe width in px (default 375 in landing generators) |
| `baseHeight` | No | integer | Mockup iframe height in px (default 820) |
| `clipBottomPx` | No | integer | Pixels clipped from iframe bottom (default 0) |
| `useOuterDeviceFrame` | No | boolean | Show device frame in landing embed |
| `notes` | No | string | Build/deploy notes |

After mockup deploy, automation should set `mockup.previewUrl` and `deployment.mockup.url` to the same live URL.

---

## media

Asset references. Each asset is an object with at least `path` (relative to package root).

| Field | Type | Notes |
|-------|------|-------|
| `icon` | mediaAsset | App icon |
| `logo` | mediaAsset | Wordmark or logo |
| `ogImage` | mediaAsset | Open Graph / social share image |
| `screenshots` | mediaAsset[] | Up to 10 screenshots |
| `demoVideo` | mediaAsset | Product demo video |

**mediaAsset:** `{ path, alt?, title?, description?, width?, height? }`

`title` and `description` are optional captions commonly used on screenshot carousel sections.

---

## landingPage

Defines landing page structure, section order, and where copy lives.

| Field | Required | Type | Notes |
|-------|----------|------|-------|
| `slug` | No | string | URL slug; defaults to `appId` |
| `sections` | Yes | section[] | Ordered list of page sections |
| `seo` | No | object | `{ title, description, keywords[] }` |

### Section object

| Field | Required | Type | Notes |
|-------|----------|------|-------|
| `id` | Yes | enum | `hero`, `features`, `screenshots`, `socialProof`, `pricing`, `faq`, `cta`, `footer` |
| `enabled` | Yes | boolean | Whether to render this section |
| `source` | Yes | enum | `inline`, `file`, or `media` |
| `inline` | If enabled + inline | object | `{ headline?, subheadline?, body?, placeholder? }` — `placeholder` for CTA email input |
| `file` | If enabled + file | string | Path to markdown (e.g. `copy/hero.md`) |

When `source` is `media`, the section pulls from `media.screenshots` — no `inline` or `file` field is needed. Example:

```json
{ "id": "screenshots", "enabled": true, "source": "media" }
```

**Minimal packages** use `source: "inline"` for `hero` and `cta` only.  
**Full packages** reference `copy/*.md` files for long-form content and may include a `screenshots` section with `source: "media"`.

### copy/ markdown conventions

Landing generators read markdown from `copy/` when sections use `source: "file"`. **App-specific content must live in the package** — generators translate package data only and must not embed app-specific defaults.

| File | Sections | Format |
|------|----------|--------|
| `copy/hero.md` | `hero` | `## Headline`, `## Subheadline`, optional `## Body` |
| `copy/benefits.md` | Hero bullets, benefit grid | `## Benefit N`, `**Title:**`, optional `**Icon:**` (`sparkles`, `check`, `star`, `zap`, `shield`, `heart`, `phone`), description paragraph |
| `copy/features.md` | `features` | `## Feature N`, `**Title:**`, description paragraph |
| `copy/faq.md` | `faq` | `## Question?` as H2, answer paragraphs below |

Benefits are not declared in `app.json` — reference `copy/benefits.md` from your package and ensure the landing generator reads it (see landing-template `generate-app-config.js`).

---

## ads

Facebook and paid social ad copy. All fields optional; populate before `validating`.

| Field | Type | Notes |
|-------|------|-------|
| `campaignName` | string | Internal campaign identifier |
| `objective` | enum | `awareness`, `traffic`, `conversions`, `leads`, `app-installs` |
| `platforms` | string[] | Ad platforms: `facebook`, `instagram`, `google`, `tiktok`, `twitter`, `linkedin` |
| `headlines` | string[] | Up to 5 variants (≤40 chars each) |
| `primaryTexts` | string[] | Up to 5 variants (≤125 chars each) |
| `descriptions` | string[] | Up to 5 variants (≤90 chars each) |
| `callToAction` | enum | `LEARN_MORE`, `SHOP_NOW`, `SIGN_UP`, `DOWNLOAD`, `GET_OFFER`, `SUBSCRIBE` |
| `utmTemplate` | object \| string | Structured UTM params (preferred) or legacy query string |

### utmTemplate (structured)

| Field | Type | Notes |
|-------|------|-------|
| `source` | string | `utm_source` value; may use `{{appId}}` |
| `medium` | string | `utm_medium` value |
| `campaign` | string | `utm_campaign` value; may use `{{appId}}` |
| `content` | string | Optional `utm_content` |
| `term` | string | Optional `utm_term` |

Example:

```json
"utmTemplate": {
  "source": "facebook",
  "medium": "paid_social",
  "campaign": "{{appId}}-validation"
}
```

The string form (e.g. `"utm_source=facebook&utm_medium=paid&utm_campaign={{appId}}"`) remains valid for backward compatibility but is deprecated.

---

## tracking

Webhook placeholders and custom event definitions for the analytics pipeline.

### webhookUrl (canonical)

| Field | Type | Notes |
|-------|------|-------|
| `webhookUrl` | string \| null | **Canonical** unified landing-event webhook. All `page_view`, `buy_now_clicked`, `email_captured`, and `mockup_interacted` events POST here. Required before `status: ready`. Provisioned during `provisioning`. |

### webhooks (legacy / pipeline)

All values are URI strings or `null` until provisioned.

| Key | Fired when |
|-----|------------|
| `validationComplete` | Package passes validation |
| `deployComplete` | Mockup and landing page are live |
| `emailCaptured` | Legacy fallback for `email_captured` landing events |
| `buyNowClicked` | Legacy fallback for `buy_now_clicked` landing events |

When `tracking.webhookUrl` is set, the landing template routes all events to it. Legacy per-event URLs are used only when `webhookUrl` is null.

### events

Optional array of `{ name, description }`. Event `name` must be snake_case.

---

## analytics

Identifiers that route experiment data to the analytics dashboard.

| Field | Type | Notes |
|-------|------|-------|
| `projectId` | string | Top-level analytics project |
| `experimentId` | string | Stable experiment family identifier |
| `experimentRunId` | string | Immutable run ID for one validation cycle |
| `landingVariantId` | string | Landing copy/layout variant (e.g. `v1`) |
| `mockupVersionId` | string | Mockup build variant (e.g. `v1`) |
| `funnelName` | string | Funnel definition name |
| `customDimensions` | object | Key-value map for dashboard filters |

`analytics.experimentId` and `experiment.experimentName` serve different purposes: the former is a machine ID for dashboards; the latter is a human-readable experiment label.

Required before `status: provisioning` — see [validator-gate.md](docs/validator-gate.md).

---

## experiment

Validation-specific data. Defines the hypothesis, budget, and rules for promoting or killing an app idea.

| Field | Required | Type | Notes |
|-------|----------|------|-------|
| `experimentName` | Yes* | string | Human-readable experiment name |
| `hypothesis` | Yes* | string | What you believe will happen and why |
| `successCriteria` | Yes* | string[] | Measurable success outcomes |
| `testBudget` | Yes* | object | See below |
| `decisionRules` | Yes* | object | See below |

\* Required before `status: provisioning` (enforced in Phase 2 validator).

### testBudget

| Field | Required | Type |
|-------|----------|------|
| `currency` | Yes | ISO 4217 string |
| `amount` | Yes | number (≥ 0) |
| `durationDays` | Yes | integer (1–90) |

### decisionRules

| Field | Type | Notes |
|-------|------|-------|
| `winnerThreshold` | string | Rule for setting `status: winner` |
| `killThreshold` | string | Rule for setting `status: killed` |
| `minSampleSize` | integer | Minimum visitors or events before deciding |
| `notes` | string | Manual review guidance |

---

## deployment

Generated and deployed URLs and platform IDs. **Populated by automation** — leave all nested fields `null` in new packages.

**v1 model:** one Vercel project per mockup, one Vercel project per landing page.

### mockup

| Field | Type | Notes |
|-------|------|-------|
| `vercelProjectId` | string \| null | Vercel project for the mockup |
| `url` | string \| null | Live mockup URL after deploy |
| `lastDeployedAt` | string \| null | ISO 8601 timestamp of most recent mockup deploy |

### landing

| Field | Type | Notes |
|-------|------|-------|
| `vercelProjectId` | string \| null | Vercel project for the landing page |
| `url` | string \| null | Canonical public landing URL (ad destination) |
| `deploymentUrl` | string \| null | Latest Vercel deployment URL |
| `lastDeployedAt` | string \| null | ISO 8601 timestamp of most recent landing deploy |

### githubRepoUrl

| Field | Type | Notes |
|-------|------|-------|
| `githubRepoUrl` | string \| null | Source repository URL if created |

n8n workflows should write these fields back to `app.json` on Google Drive after deploy steps complete. On mockup deploy, also set `mockup.previewUrl` to match `deployment.mockup.url`.

---

## appStore

Reserved for future App Store metadata workflows. Optional in Phase 1.

| Field | Type | Notes |
|-------|------|-------|
| `subtitle` | string | App Store subtitle (≤30 chars) |
| `keywords` | string[] | Search keywords (≤10) |
| `promotionalText` | string | Promotional blurb (≤170 chars) |
| `description` | string | Full store description (≤4000 chars) |
| `appStoreUrl` | string \| null | Live App Store URL after publish |
| `supportUrl` | string | Support page URL |
| `privacyPolicyUrl` | string | Privacy policy URL |

---

## Validation profiles

### Minimal profile

Smallest valid package for early ideation:

- Root: `specVersion`, `appId`, `status` (`draft`), `identity`, `audience`, `commerce`, `landingPage`
- `landingPage.sections`: at least `hero` and `cta` enabled with `source: "inline"`
- No `copy/`, `media/`, `mockup/`, `ads/`, `deployment`, or `appStore` required

### Ready profile

Package ready for automation:

- All minimal requirements
- Full `experiment` section populated
- Analytics IDs: `projectId`, `experimentId`, `experimentRunId`, `landingVariantId`, `mockupVersionId`
- `status` set to `ready` (requires `tracking.webhookUrl` — typically set during `provisioning`)
- Recommended: `branding`, `mockup`, `media`, `ads`, `tracking`, `analytics`

See [docs/validator-gate.md](docs/validator-gate.md) for the full gate checklist.

### Full profile

Reference package using every section, including `appStore`, file-based copy (`hero`, `benefits`, `features`, `faq`), and landing-specific fields (`badgeText`, `landingPhrase`, `landingStyle`, mockup embed dimensions).

---

## Landing page transform

Consumers (e.g. `landing-template/scripts/generate-app-config.js`) map App Package data into landing `app-config.json`. Rules:

1. **Single source of truth:** App-specific copy and labels come from `app.json` and `copy/` — not from generator hardcoded defaults.
2. **Generic fallbacks only:** When a field is omitted, generators may use platform-neutral defaults (e.g. empty string, `clipBottomPx: 0`, platform-based badge for iOS).
3. **Key mappings:** `audience.landingPhrase` → target audience line; `identity.badgeText` → hero badge; `copy/benefits.md` → `benefits[]`; `landingPage.seo` → page metadata; `landingPage.sections[footer].inline.body` → footer text; `commerce.cta.waitlistText` → email capture button; CTA `inline.placeholder` → email input placeholder.

See your landing template's transform documentation for the full field reference.

---

## Versioning

- `specVersion` in every `app.json` must match a published spec version.
- Breaking changes increment the major version; new optional fields increment minor.
- See [CHANGELOG.md](CHANGELOG.md) and [docs/versioning.md](docs/versioning.md) for migration guidance.

---

## Template conventions

The file [templates/app.json](templates/app.json) includes `_comment` keys for documentation. **Real App Packages must not include `_comment` keys** — they are not part of the schema and will fail validation in Phase 2.

---

## Out of scope (Phase 1)

- Validator CLI or CI enforcement
- n8n workflow definitions
- Mockup deploy tooling

Landing page generators consume this spec; they must translate package data rather than owning app-specific content.
