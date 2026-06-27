# App Validation Spec

Official **App Package Specification** for the automated app validation system.

This repository is the source-of-truth contract that humans, AI tools, validators, and n8n workflows follow when creating, validating, and deploying app ideas.

## What is an App Package?

An **App Package** is a folder of structured files that describes a single app idea end to end: identity, audience, pricing, landing page copy, mockup source, ads, tracking, experiment design, and deployment outputs.

The canonical manifest is **`app.json`**.

## Pipeline (future)

This spec supports an automated pipeline that will eventually:

1. Read a standardized app package from Google Drive
2. Validate the package
3. Deploy the interactive mockup app
4. Generate a premium landing page
5. Deploy the landing page to Vercel
6. Create Facebook ads
7. Track emails and Buy Now clicks
8. Feed results into an analytics dashboard

**Phase 1** defines documentation, schema, templates, and examples. No validator, n8n workflows, or landing page generator are included yet.

## Quick start

1. Read [APP_PACKAGE_SPEC.md](APP_PACKAGE_SPEC.md) for the full field reference.
2. Copy [templates/app.json](templates/app.json) into a new `{appId}/` folder as `app.json`.
3. Optionally add [templates/copy/](templates/copy/) markdown files for long-form landing copy.
4. Compare your package to [examples/minimal-app/](examples/minimal-app/) (smallest valid) or [examples/full-app/](examples/full-app/) (complete reference).
5. Validate against [schemas/app.schema.json](schemas/app.schema.json) when the Phase 2 validator is available.

## Repository structure

```txt
app-validation-spec/
├── README.md
├── APP_PACKAGE_SPEC.md
├── CHANGELOG.md
├── schemas/
│   └── app.schema.json
├── templates/
│   ├── app.json
│   └── copy/
│       ├── hero.md
│       ├── features.md
│       └── faq.md
├── examples/
│   ├── minimal-app/
│   └── full-app/
└── docs/
    ├── design-philosophy.md
    ├── workflow.md
    ├── n8n-integration-notes.md
    ├── naming-conventions.md
    └── versioning.md
```

## Key documents

| Document | Purpose |
|----------|---------|
| [APP_PACKAGE_SPEC.md](APP_PACKAGE_SPEC.md) | Normative human-readable contract |
| [schemas/app.schema.json](schemas/app.schema.json) | Machine-readable JSON Schema |
| [CHANGELOG.md](CHANGELOG.md) | Spec version history |
| [docs/design-philosophy.md](docs/design-philosophy.md) | Design decisions and tradeoffs |
| [docs/workflow.md](docs/workflow.md) | Pipeline stages mapped to spec fields |
| [docs/n8n-integration-notes.md](docs/n8n-integration-notes.md) | Drive layout, parsing, write-back |
| [docs/naming-conventions.md](docs/naming-conventions.md) | `appId`, paths, filenames |
| [docs/versioning.md](docs/versioning.md) | Semver policy and migrations |

## Examples

| Example | App | Profile |
|---------|-----|---------|
| [examples/minimal-app/](examples/minimal-app/) | Focus Timer | Single `app.json`, inline copy, `status: draft` |
| [examples/full-app/](examples/full-app/) | Habit Stack | All sections, file-based copy, `status: ready` |

## Spec version

Current spec version: **1.0.0**

Every `app.json` must include `"specVersion": "1.0.0"` until a new version is published. See [docs/versioning.md](docs/versioning.md).

## Contributing

When proposing spec changes:

1. Update [schemas/app.schema.json](schemas/app.schema.json) and [APP_PACKAGE_SPEC.md](APP_PACKAGE_SPEC.md) together.
2. Update [templates/app.json](templates/app.json) and relevant [examples/](examples/).
3. Add a [CHANGELOG.md](CHANGELOG.md) entry and bump `specVersion` per [docs/versioning.md](docs/versioning.md).
4. Update supporting docs in [docs/](docs/) if behavior or conventions change.

## License

TBD.
