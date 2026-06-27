# Mockup source

This folder holds the interactive mockup source code referenced by `app.json` → `mockup`.

## Expected layout

```txt
mockup/
├── index.html          # Entry point (or framework build output)
├── package.json        # If using a JS framework
└── ...                 # Source files
```

## Configuration in app.json

The full-app example declares:

| Field | Value |
|-------|-------|
| `type` | `interactive` |
| `sourcePath` | `mockup/` |
| `framework` | `react` |
| `entryPoint` | `mockup/index.html` |
| `buildCommand` | `npm run build` |
| `deployCommand` | `npx vercel deploy --prod` |
| `previewUrl` | `null` (set by automation after deploy) |

## Phase 1 note

No deployable mockup code is included in this example. In later phases, n8n will read `buildCommand` and `deployCommand`, deploy the mockup, and write back `mockup.previewUrl` and `deployment.mockupUrl`.

## Getting started

1. Scaffold a simple interactive prototype in this folder
2. Ensure `entryPoint` resolves after `buildCommand` completes
3. Test locally before setting `status` to `ready`
