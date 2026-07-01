# Minimal App Package Example

This example demonstrates the **smallest valid App Package** for early ideation.

## What makes it minimal

- Single file: `app.json` only (no `copy/`, `media/`, or `mockup/` folders)
- `status` is `draft` — experiment and deployment sections are omitted
- Landing page uses **inline copy** for `hero` and `cta` sections only
- No optional sections: `branding`, `mockup`, `media`, `ads`, `tracking`, `analytics`, `experiment`, `deployment`, or `appStore`
- No `screenshots` section — that requires `media.screenshots` and `source: "media"`

The minimal profile intentionally omits `deployment`, `tracking`, `analytics`, and `experiment`. That is valid for early ideation only — expand to the [full-app example](../full-app/) before setting `status: provisioning`.

## App idea

**Focus Timer** — a one-time-purchase iOS productivity app for timed deep-work sessions.

## When to use this profile

Use a minimal package when you are capturing a raw app idea before investing in copy files, media assets, or experiment design. Expand to the [full-app example](../full-app/) when you are ready to run validation.

## Validate

In Phase 2, validate against `schemas/app.schema.json` from the repo root:

```bash
# Example (validator not yet implemented)
# validate-app examples/minimal-app/app.json
```

This example uses `specVersion` **1.3.0**. Packages at older versions remain valid if they omit newer optional fields — see [docs/versioning.md](../../docs/versioning.md).
