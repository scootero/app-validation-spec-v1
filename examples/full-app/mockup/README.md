# Mockup source

This folder holds the interactive mockup source code referenced by `app.json` → `mockup`.

## Expected layout

```txt
mockup/
├── package.json
├── src/
│   ├── App.jsx
│   └── main.jsx
└── ...
```

## Configuration in app.json

The full-app example declares:

| Field | Value |
|-------|-------|
| `type` | `interactive` |
| `sourcePath` | `mockup/` |
| `framework` | `react-vite` |
| `entryPoint` | `mockup/src/App.jsx` |
| `installCommand` | `npm install` |
| `buildCommand` | `npm run build` |
| `devCommand` | `npm run dev` |
| `deployCommand` | `npx vercel deploy --prod` |
| `previewUrl` | `null` (set by automation after deploy) |

## Phase 1 note

No deployable mockup code is included in this example. In later phases, n8n will read `installCommand`, `buildCommand`, and `deployCommand`, deploy the mockup, and write back `mockup.previewUrl`, `deployment.mockup.url`, `deployment.mockup.vercelProjectId`, and `deployment.mockup.lastDeployedAt`.

## Getting started

1. Scaffold a simple interactive prototype in this folder
2. Run `npm install` and `npm run dev` to preview locally
3. Ensure `entryPoint` resolves after `buildCommand` completes
4. Test locally before setting `status` to `provisioning` (then `ready` after `tracking.webhookUrl` is provisioned)
