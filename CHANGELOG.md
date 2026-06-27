# Changelog

All notable changes to the App Package Specification are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and spec versioning follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-06-26

### Added

- Initial App Package Specification (`specVersion` 1.0.0).
- JSON Schema for `app.json` at `schemas/app.schema.json`.
- Root fields: `specVersion`, `appId`, `status`.
- Core sections: `identity`, `audience`, `commerce`, `branding`, `mockup`, `media`, `landingPage`, `ads`, `tracking`, `analytics`.
- Validation sections: `experiment` (hypothesis, budget, decision rules), `deployment` (generated URLs and platform IDs).
- Reserved optional section: `appStore` for future App Store metadata.
- `status` lifecycle enum: `draft`, `ready`, `validating`, `paused`, `winner`, `killed`, `built`.
- `mockup` fields for source project (`sourcePath`, `framework`, `entryPoint`, `buildCommand`, `deployCommand`) and deployed preview (`previewUrl`).
- Starter template at `templates/app.json`.
- Copy templates at `templates/copy/` (hero, features, faq).
- Example packages: `examples/minimal-app/` (Focus Timer), `examples/full-app/` (Habit Stack).
- Supporting documentation in `docs/` (design philosophy, workflow, n8n notes, naming, versioning).

[1.0.0]: https://github.com/app-validation-spec/app-validation-spec/releases/tag/v1.0.0
