# Full App Package Example

This example demonstrates a **complete, automation-ready App Package** using every section defined in spec version 1.0.0.

## What makes it full

- All top-level `app.json` sections populated
- File-based landing copy in `copy/` (hero, features, faq)
- `media/` and `mockup/` folders with README placeholders (no binaries or source code in this repo)
- `experiment` fully defined with hypothesis, budget, and decision rules
- `status` set to `ready` — eligible for validation kickoff
- `deployment` and webhook fields present but `null` until automation runs

## App idea

**Habit Stack** — a subscription cross-platform wellness app for building daily habit routines with stacking, streaks, and weekly insights.

## Folder layout

```txt
habit-stack/
├── app.json
├── copy/
│   ├── hero.md
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
2. Add mockup source to `mockup/` and verify `buildCommand` works locally
3. Confirm `experiment` success criteria and budget with stakeholders
4. Set `status` to `ready` (already set in this example)
