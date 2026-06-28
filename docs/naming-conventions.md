# Naming Conventions

Consistent naming keeps App Packages predictable across Google Drive, URLs, analytics, and automation.

## appId

The `appId` is the primary identifier for an app idea.

### Format

- **kebab-case**: lowercase letters, digits, and hyphens only
- **Pattern:** `^[a-z][a-z0-9]*(-[a-z0-9]+)*$`
- **Length:** 2–64 characters
- **Must not** start or end with a hyphen
- **Must not** contain consecutive hyphens (avoid `--`)

### Examples

| Valid | Invalid | Reason |
|-------|---------|--------|
| `focus-timer` | `FocusTimer` | No uppercase |
| `habit-stack` | `habit_stack` | No underscores |
| `pay-flow-v2` | `2pay-flow` | Must start with a letter |
| `ai-writer` | `ai--writer` | No double hyphens |

### Usage

| Context | Value |
|---------|-------|
| Google Drive folder name | `{appId}/` |
| `app.json` → `appId` | Must match folder name |
| Default landing slug | `appId` (unless `landingPage.slug` overrides) |
| Analytics project ID prefix | `proj_{appId}` (convention) |
| Experiment ID prefix | `exp_{appId}_{suffix}` (convention) |
| Ad campaign name | Often `{appId}-launch` or value in `ads.campaignName` |

### Immutability

**Never rename `appId` after first deploy.** Create a new package for a genuinely new idea.

## appName

Display name shown to users. Rules:

- 1–80 characters
- Any casing and spacing allowed
- Can change without breaking automation (unlike `appId`)

## Folder structure

```
{appId}/
├── app.json
├── copy/
├── media/
│   └── screenshots/
├── mockup/
└── README.md
```

- Folder name **must** equal `appId`
- Subfolder names are lowercase: `copy`, `media`, `mockup`
- No spaces in folder or file names

## File naming

### app.json

- Always `app.json` at package root
- Never `App.json`, `app.JSON`, or nested paths

### Copy files

| File | Purpose |
|------|---------|
| `copy/hero.md` | Hero section |
| `copy/features.md` | Features section |
| `copy/faq.md` | FAQ section |

Use lowercase filenames. Additional copy files are allowed if referenced from `landingPage.sections[].file`.

### Media files

| File | Convention |
|------|------------|
| `media/icon.png` | App icon |
| `media/logo.png` | Logo / wordmark |
| `media/og-image.png` | Open Graph image (kebab-case) |
| `media/demo.mp4` | Demo video |
| `media/screenshots/01-home.png` | Numbered screenshots |

Screenshot metadata (`title`, `description`, `alt`) is declared in `app.json` → `media.screenshots[]`, not in filenames.

Screenshot file rules:

- Two-digit numeric prefix: `01-`, `02-`, …
- Descriptive kebab-case suffix after prefix
- PNG preferred for screenshots; SVG allowed for logos

### Mockup files

| File | Convention |
|------|------------|
| `mockup/src/App.jsx` | Common entry point for React/Vite mockups |
| `mockup/index.html` | Static HTML entry point |
| `mockup/package.json` | If using a JS toolchain |

Declare actual entry in `mockup.entryPoint`. Use `installCommand` and `devCommand` for local setup; `buildCommand` and `deployCommand` for automation.

## landingPage.slug

- Same pattern as `appId` when customized
- Defaults to `appId` if omitted
- Used in Vercel URL paths: `https://{domain}/{slug}`

## Analytics identifiers

Conventions (not schema-enforced):

| Field | Pattern | Example |
|-------|---------|---------|
| `analytics.projectId` | `proj_{appId}` | `proj_habit-stack` |
| `analytics.experimentId` | `exp_{appId}_{period}_{seq}` | `exp_habit-stack_2026q2_001` |
| `analytics.funnelName` | snake_case | `validation_funnel` |

## Tracking event names

`tracking.events[].name` must be **snake_case**:

- `page_view`
- `email_captured`
- `buy_now_clicked`
- `trial_started`

No spaces, no hyphens, no uppercase.

## ads.campaignName

- Lowercase kebab-case or snake_case recommended
- Should be unique per validation run
- Example: `habit-stack-q2-validation`

## ads.platforms

Use lowercase enum values from the schema:

- `facebook`, `instagram`, `google`, `tiktok`, `twitter`, `linkedin`

List only platforms you intend to run ads on.

## ads.utmTemplate

Prefer the structured object form (since 1.1.0):

```json
{
  "source": "facebook",
  "medium": "paid_social",
  "campaign": "{{appId}}-validation",
  "content": "{{headline_variant}}"
}
```

Legacy string form is still valid: `"utm_source=facebook&utm_medium=paid_social&utm_campaign={{appId}}-validation"`.

Placeholders: `{{appId}}`, `{{headline_variant}}` (expand in n8n before ad creation).

## experiment.experimentName

Human-readable — any casing allowed:

- `Habit Stack — Q2 2026 Validation`
- `Focus Timer MVP Test`

## specVersion

- Strict semver: `MAJOR.MINOR.PATCH`
- Current: `1.1.0`
- Do not use pre-release tags in `app.json` (`1.0.0-beta` is invalid)

## Template _comment keys

`templates/app.json` uses `_comment` keys for documentation.

**Real App Packages must not include `_comment` keys.** They are not part of the schema and will fail Phase 2 validation.

## Related documents

- [APP_PACKAGE_SPEC.md](../APP_PACKAGE_SPEC.md) — field definitions and patterns
- [versioning.md](versioning.md) — spec version rules
- [n8n-integration-notes.md](n8n-integration-notes.md) — Drive folder discovery
