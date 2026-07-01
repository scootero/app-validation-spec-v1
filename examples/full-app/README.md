# Full App Package Example

This example demonstrates a **complete, automation-ready App Package** using every section defined in spec version 1.3.0.

## What makes it full

- All top-level `app.json` sections populated
- File-based landing copy in `copy/` (hero, benefits, features, faq)
- **Screenshots section** with `source: "media"` — pulls from `media.screenshots` with title and description captions
- `media/` and `mockup/` folders with README placeholders (no binaries or source code in this repo)
- `experiment` fully defined with hypothesis, budget, and decision rules
- Structured `ads.utmTemplate` and `ads.platforms`
- Analytics IDs: `projectId`, `experimentId`, `experimentRunId`, `landingVariantId`, `mockupVersionId`
- `status` set to `provisioning` — package complete; awaiting `tracking.webhookUrl` before `ready`
- Nested `deployment.mockup.*` and `deployment.landing.*` fields present but `null` until automation runs

## App idea

**Habit Stack** — a subscription cross-platform wellness app for building daily habit routines with stacking, streaks, and weekly insights.

## Folder layout

```txt
habit-stack/
├── app.json
├── copy/
│   ├── hero.md
│   ├── benefits.md
│   ├── features.md
│   └── faq.md
├── media/
│   └── README.md
├── mockup/
│   └── README.md
└── README.md
```

## Compare with minimal

See [minimal-app](../minimal-app/) for the smallest valid package — single `app.json` with inline hero and cta only.

## Validate

In Phase 2, validate against `schemas/app.schema.json` from the repo root:

```bash
# Example (validator not yet implemented)
# validate-app examples/full-app/app.json
```

## Before running ads

1. Add real files to `media/` matching paths in `app.json`
2. Add mockup source to `mockup/` and verify `installCommand`, `buildCommand`, and `devCommand` work locally
3. Confirm `experiment` success criteria and budget with stakeholders
4. Set `status` to `provisioning` when the package is complete
5. After n8n provisions `tracking.webhookUrl`, set `status` to `ready`
