# Media assets

This folder holds image and video files referenced by `app.json` → `media`.

## Expected files

| File | Purpose | Recommended size |
|------|---------|------------------|
| `icon.png` | App icon | 1024×1024 px |
| `logo.png` | Wordmark / logo | SVG or PNG, transparent background |
| `og-image.png` | Social share / Open Graph image | 1200×630 px |
| `screenshots/01-home.png` | Home screen screenshot | Device-native resolution |
| `screenshots/02-feature.png` | Key feature screenshot | Device-native resolution |
| `screenshots/03-pricing.png` | Pricing or trial screen | Device-native resolution |
| `demo.mp4` | Optional product demo video | ≤ 60 seconds |

## Screenshot captions

Declare optional `title` and `description` on each entry in `media.screenshots` in `app.json`. The landing page `screenshots` section (`source: "media"`) uses those captions when rendering the carousel.

## Phase 1 note

This example package documents expected filenames only. Real image binaries are not included in the spec repository to keep it lightweight. Add actual assets to your App Package on Google Drive before running validation with ads or landing page generation.

## Naming conventions

- Use lowercase kebab-case filenames
- Prefix screenshots with two-digit order: `01-`, `02-`, etc.
- Match paths exactly as declared in `app.json`
