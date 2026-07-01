# Validator Gate

Documented pre-deploy validation checks for App Packages. The Phase 2 validator CLI will enforce these rules; this document is the normative reference until that tool exists.

See also: [workflow.md](workflow.md), [APP_PACKAGE_SPEC.md](../APP_PACKAGE_SPEC.md) validation profiles.

## Lifecycle gates

| Transition | Required before promotion |
|------------|-------------------------|
| `draft` → `provisioning` | Full `experiment` section; complete `ads`; analytics IDs; copy/media/mockup present per package profile |
| `provisioning` → `ready` | `tracking.webhookUrl` is a non-null URI (provisioned by n8n) |
| `ready` → pipeline | Passes all checks below |

## Checks (by category)

### Schema

- `app.json` validates against `schemas/app.schema.json` at the package `specVersion`
- `appId` matches folder name on Google Drive

### Copy

- Every `landingPage.sections[]` entry with `source: "file"` has a resolvable file on disk
- `copy/benefits.md` exists when the landing template expects benefits (recommended for full packages)

### Media

- Every path in `media.screenshots`, `media.logo`, `media.icon`, and `media.ogImage` resolves to a file — or an explicit placeholder policy is documented for the package

### Mockup

- When `mockup` section is present: `mockup.buildCommand` succeeds in CI/automation
- `mockup.entryPoint` and `mockup.sourcePath` resolve

### Landing generation

- `landing-template/scripts/generate-app-config.js <package-path>` produces valid `app-config.json` without errors

### Analytics (before `provisioning`)

| Field | Required |
|-------|----------|
| `analytics.projectId` | Yes |
| `analytics.experimentId` | Yes |
| `analytics.experimentRunId` | Yes |
| `analytics.landingVariantId` | Yes (default `v1`) |
| `analytics.mockupVersionId` | Yes (default `v1`) |

### Webhook (before `ready`)

- `tracking.webhookUrl` must be a non-null URI
- Legacy `tracking.webhooks.emailCaptured` and `tracking.webhooks.buyNowClicked` are optional fallbacks only

### Deployment

- Before deploy: nested `deployment.mockup.*` and `deployment.landing.*` fields are `null`
- After mockup deploy: `deployment.mockup.url`, `deployment.mockup.vercelProjectId`, `deployment.mockup.lastDeployedAt` populated; `mockup.previewUrl` matches `deployment.mockup.url`
- After landing deploy: `deployment.landing.url`, `deployment.landing.vercelProjectId`, `deployment.landing.deploymentUrl`, `deployment.landing.lastDeployedAt` populated

### Experiment (before `provisioning`)

Full `experiment` section required:

- `experimentName`, `hypothesis`, `successCriteria`
- `testBudget` (currency, amount, durationDays)
- `decisionRules` (winnerThreshold, killThreshold, minSampleSize)

## On failure

- Do not deploy, provision ads, or spend budget
- Notify author with section-scoped errors
- Leave `status` unchanged or revert to `draft`

## On success

- `provisioning` workflow: write `tracking.webhookUrl`, set `status: ready`
- Main pipeline: proceed to mockup deploy → landing generate → landing deploy → ads
