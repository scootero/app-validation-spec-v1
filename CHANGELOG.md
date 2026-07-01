# Changelog

All notable changes to the App Package Specification are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and spec versioning follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.3.0] - 2026-06-30

### Added

- `status: provisioning` — intermediate lifecycle state for webhook provisioning before `ready`.
- `tracking.webhookUrl` — canonical unified landing-event webhook (required before `ready`).
- `analytics.experimentRunId`, `analytics.landingVariantId`, `analytics.mockupVersionId` for dashboard and A/B attribution.
- Nested `deployment.mockup` and `deployment.landing` objects for v1 deploy model (one Vercel project per mockup and per landing).
- [docs/validator-gate.md](docs/validator-gate.md) — documented pre-deploy validation checks (Phase 2 CLI not yet implemented).

### Changed

- **Breaking:** Flat `deployment` fields replaced by nested structure. Migration:

| 1.2.0 | 1.3.0 |
|-------|-------|
| `deployment.mockupUrl` | `deployment.mockup.url` |
| `deployment.landingPageUrl` | `deployment.landing.url` |
| `deployment.vercelProjectId` | `deployment.landing.vercelProjectId` |
| `deployment.vercelDeploymentUrl` | `deployment.landing.deploymentUrl` |
| `deployment.lastDeployedAt` | `deployment.landing.lastDeployedAt` |

- Landing generators should read nested deployment first; flat fields may be supported temporarily as fallbacks for unmigrated packages.
- `tracking.webhooks.emailCaptured` and `tracking.webhooks.buyNowClicked` remain optional legacy fallbacks; `tracking.webhookUrl` is canonical.

### Deprecated

- Flat `deployment.mockupUrl`, `deployment.landingPageUrl`, `deployment.vercelProjectId`, `deployment.vercelDeploymentUrl`, `deployment.lastDeployedAt` (removed from schema; use nested fields).

## [1.2.0] - 2026-06-29

### Added

- `copy/benefits.md` convention for short value props (hero bullets and benefit grid).
- `identity.badgeText` for hero badge label on landing pages.
- `audience.landingPhrase` for short *Built for …* audience line.
- `branding.theme.landingStyle` — landing theme preset enum (`liquid-glass`, `midnight`, etc.).
- `branding.theme.accentName` — landing accent palette name (`violet`, `emerald`, etc.).
- `mockup.baseWidth`, `mockup.baseHeight`, `mockup.clipBottomPx`, `mockup.useOuterDeviceFrame` for landing embed sizing.
- `commerce.pricing.headlineLabel` for pricing section label before amount.
- `commerce.cta.emailPlaceholder` for email capture input placeholder.
- `landingPage.sections[].inline.placeholder` for CTA email input (preferred over `emailPlaceholder`).
- [APP_PACKAGE_SPEC.md](APP_PACKAGE_SPEC.md) landing transform section: App Package is single source of truth; generators translate only.

### Changed

- Starter template and full example updated to `specVersion` 1.2.0 with new landing fields demonstrated.
- `templates/copy/benefits.md` added; `examples/full-app/copy/benefits.md` added.

## [1.1.0] - 2026-06-27

### Added

- `landingPage.sections`: new section id `screenshots` with `source: "media"` to render from `media.screenshots`.
- `mockup.installCommand` and `mockup.devCommand` for dependency install and local dev.
- `deployment.vercelDeploymentUrl` and `deployment.lastDeployedAt` for Vercel deploy tracking.
- `appStore.supportUrl` and `appStore.privacyPolicyUrl` for store listing metadata.
- `mediaAsset.title` and `mediaAsset.description` optional caption fields for screenshots.
- `ads.platforms` array for ad platform selection.
- Structured `ads.utmTemplate` object (`source`, `medium`, `campaign`, `content`, `term`); string form retained for backward compatibility.

### Changed

- Starter template updated to `specVersion` 1.1.0 with new fields demonstrated.

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

[1.2.0]: https://github.com/app-validation-spec/app-validation-spec/releases/tag/v1.2.0
[1.1.0]: https://github.com/app-validation-spec/app-validation-spec/releases/tag/v1.1.0
[1.0.0]: https://github.com/app-validation-spec/app-validation-spec/releases/tag/v1.0.0
