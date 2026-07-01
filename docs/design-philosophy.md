# Design Philosophy

This document explains the design decisions behind the App Package Specification and the tradeoffs we accepted.

## Core idea

Every app idea becomes an **App Package**: a folder of structured files that humans can understand and automation can reliably consume. The manifest is always `app.json`.

The spec is the contract between four audiences:

| Audience | Need |
|----------|------|
| Humans | Clear fields, sensible defaults, minimal required set for early ideas |
| AI generators | Predictable structure, enums with escape hatches, copy separated from config |
| n8n workflows | Flat dot-notation paths, stable keys, nullable automation write-backs |
| Validators (Phase 2+) | JSON Schema, explicit required fields, version pinning via `specVersion` |

## Why JSON for app.json

YAML and TOML are easier to hand-edit, but JSON wins for this system:

- n8n parses JSON natively with minimal transformation
- JSON Schema validation is mature and tooling is widespread
- AI models generate valid JSON reliably when given a schema
- No ambiguity around types (numbers vs strings)

Long-form prose lives in `copy/*.md`, not inside JSON strings, so the manifest stays scannable.

## Why inline and file-based copy coexist

Landing page content can be declared three ways via `landingPage.sections[].source`:

| Source | Best for |
|--------|----------|
| `inline` | Minimal packages, quick drafts, short hero/cta blocks |
| `file` | Full packages, AI-generated markdown, collaborative copy editing |
| `media` | Screenshot carousels — pulls from `media.screenshots` with optional title/description captions |

This avoids forcing a `copy/` folder on every idea while keeping full packages from bloating `app.json`. Screenshot paths and captions live in `media.screenshots`; the landing page only references them with `{ "id": "screenshots", "source": "media" }`.

## Section grouping

`app.json` uses named top-level sections (`identity`, `commerce`, `experiment`, etc.) rather than a flat key namespace:

- Humans navigate by concept, not alphabet
- n8n still accesses flat paths: `$json.commerce.pricing.amount`
- Validators can scope errors to a section
- New fields land in the right section without renaming existing keys

## Strictness boundaries

### Strict where it matters

- Root scalars: `specVersion`, `appId`, `status`
- Core authoring sections: `identity`, `audience`, `commerce`, `landingPage`
- Enums for `status`, `platform`, `pricing.model`, section `id`, section `source`
- `additionalProperties: false` on section objects to catch typos early

### Flexible where categories diverge

- `identity.category` enum plus `"other"` + `categoryLabel`
- Optional sections omitted entirely in minimal packages
- `analytics.customDimensions` as an open key-value map
- `tracking.events` for custom event definitions
- `appStore` reserved but optional until App Store workflows exist

### Documented but not schema-enforced (yet)

- `experiment` required before `status: provisioning` — enforced in Phase 2 validator, not JSON Schema alone, so drafts stay valid while incomplete
- `tracking.webhookUrl` required before `status: ready` — provisioned during `provisioning`
- At least `hero` + `cta` enabled in `landingPage` — documented in profiles, validated in Phase 2

## experiment vs analytics

These sections are deliberately separate:

| Section | Purpose | Who writes it |
|---------|---------|---------------|
| `experiment` | Hypothesis, budget, decision rules | Human / AI at authoring time |
| `analytics` | Dashboard IDs, funnel names, dimensions | Human at setup; n8n may extend |

`experiment.experimentName` is human-readable. `analytics.experimentId` is a machine identifier for dashboards. They relate by convention, not a hard foreign key.

## deployment and mockup.previewUrl

**v1 model:** one Vercel project per mockup, one Vercel project per landing page.

Automation writes back URLs and platform IDs after deploy steps:

- `deployment.mockup.url` and `mockup.previewUrl` should match after mockup deploy
- `deployment.landing.url` is the canonical public landing URL (ad destination)
- `deployment.landing.deploymentUrl` records the latest Vercel deployment URL
- `deployment.mockup.vercelProjectId` and `deployment.landing.vercelProjectId` link to infrastructure
- `deployment.githubRepoUrl` links to source repo if created
- `deployment.mockup.lastDeployedAt` and `deployment.landing.lastDeployedAt` are ISO 8601 deploy timestamps

Mockup authoring uses `installCommand` and `devCommand` for local setup and preview; `buildCommand` and `deployCommand` run in automation.

All deployment nested fields are nullable in new packages. n8n owns write-back; humans should not hand-edit these except for recovery.

## status as the pipeline state machine

`status` is a root-level enum so n8n can branch on a single field without inferring state from nullable URLs or webhook history.

```
draft → provisioning → ready → validating → winner → built
                              ↓           ↓
                           paused      killed
```

- `provisioning`: n8n provisions `tracking.webhookUrl`, then promotes to `ready`
- Automation may set `provisioning`, `validating`, `winner`, and `killed`
- Humans set `draft`, `paused`, and `built`

## appId immutability

`appId` is the folder name on Google Drive, the URL slug default, the analytics key, and the ad campaign anchor. Renaming it breaks links, historical data, and deployed resources.

`appName` and `landingPage.slug` can change; `appId` cannot after first deploy.

## Versioning philosophy

- `specVersion` in every `app.json` pins the contract
- Breaking changes increment major; new optional fields increment minor
- The repo and the spec version are related but not identical — see [versioning.md](versioning.md)

## ads.utmTemplate: string vs object

Spec 1.1.0 introduced a structured `ads.utmTemplate` object (`source`, `medium`, `campaign`, `content`, `term`). The legacy string form remains valid for backward compatibility. New packages should prefer the object — it is easier for n8n to expand and less error-prone than parsing query strings.

## What we explicitly deferred

| Deferred | Reason |
|----------|--------|
| Validator CLI | Phase 2 — spec must stabilize first |
| n8n workflow JSON | Phase 3+ — depends on validator and deploy tooling |
| Landing page generator | Consumer of this spec, not part of it |
| Real binaries in examples | Keeps the spec repo lightweight; paths are documented |

## Extension points

Future additions should prefer:

1. New optional fields inside existing sections
2. New optional top-level sections (with a minor `specVersion` bump)
3. New enum values only when automation truly needs them

Avoid renaming keys or changing enum meanings within the same major version.

## Related documents

- [APP_PACKAGE_SPEC.md](../APP_PACKAGE_SPEC.md) — normative field reference
- [workflow.md](workflow.md) — pipeline stages mapped to spec fields
- [n8n-integration-notes.md](n8n-integration-notes.md) — parsing and write-back conventions
- [naming-conventions.md](naming-conventions.md) — `appId`, paths, filenames
- [validator-gate.md](validator-gate.md) — pre-deploy validation checklist
- [versioning.md](versioning.md) — semver policy and migrations
