# n8n Integration Notes

Practical guidance for wiring n8n workflows against the App Package Specification. Phase 1 defines the contract only — workflow JSON is out of scope.

## Google Drive layout

### Parent folder

Configure a single parent folder for all app packages, e.g.:

```txt
App Validation/
└── {appId}/
    ├── app.json
    ├── copy/
    ├── media/
    └── mockup/
```

### Discovery

1. List child folders of the parent directory
2. For each folder, read `{folderName}/app.json`
3. Confirm `app.json.appId === folderName` — mismatch is a validation warning
4. Branch on `status`:
   - `ready` → validation pipeline entry
   - `validating` → monitoring / decision workflows
   - `draft`, `paused`, `winner`, `killed`, `built` → no automatic processing unless explicitly configured

### Polling vs webhooks

Drive does not push file-change events to n8n natively. Common patterns:

- **Schedule trigger** — poll parent folder every N minutes for `status: ready`
- **Manual trigger** — operator kicks off a specific `appId`
- **Drive webhook** (if available in your stack) — filter on parent folder path

## Reading app.json

### Parse

Use n8n's **Extract from File** or **Code** node:

```javascript
const app = JSON.parse($input.first().binary.data.toString('utf8'));
return [{ json: app }];
```

### Common field paths

| Path | Use |
|------|-----|
| `$json.appId` | Correlation ID across all nodes |
| `$json.status` | Pipeline branching |
| `$json.specVersion` | Schema version selection |
| `$json.identity.appName` | Display name in notifications |
| `$json.commerce.pricing` | Landing page pricing block |
| `$json.commerce.cta.buyNowText` | CTA button label |
| `$json.landingPage.sections` | Section order and copy sources |
| `$json.experiment.testBudget.amount` | Ad spend cap |
| `$json.ads.platforms` | Target ad platforms (e.g. facebook, instagram) |
| `$json.ads.headlines` | Ad creative variants |
| `$json.analytics.experimentId` | Dashboard event tagging |

### Loading file-based copy

For each section where `source === "file"` and `enabled === true`:

1. Read `$json.landingPage.sections[i].file` relative to package root
2. Load markdown from Drive, e.g. `copy/hero.md`
3. Pass content to landing page generator node

For each section where `source === "media"` and `enabled === true`:

1. Confirm `id === "screenshots"` (only supported media section in 1.1.0)
2. Read `$json.media.screenshots` array — no separate file path on the section
3. Download each screenshot binary from Drive using `path`
4. Pass `path`, `alt`, `title`, and `description` to the landing page generator

### Loading media

For each asset in `media`:

```javascript
const iconPath = $json.media?.icon?.path; // e.g. "media/icon.png"
```

Download binary from Drive before upload to ad platforms or static site build.

## Write-back conventions

Automation **writes back** to `app.json` on Drive after deploy and decision steps. Use merge semantics — update only changed keys, preserve author content.

### After mockup deploy

```json
{
  "mockup": {
    "previewUrl": "https://mockup-focus-timer.vercel.app"
  },
  "deployment": {
    "mockupUrl": "https://mockup-focus-timer.vercel.app"
  }
}
```

`mockup.previewUrl` and `deployment.mockupUrl` should always match.

### After landing page deploy

```json
{
  "deployment": {
    "landingPageUrl": "https://focus-timer.vercel.app",
    "vercelProjectId": "prj_abc123",
    "vercelDeploymentUrl": "https://focus-timer-abc123.vercel.app",
    "githubRepoUrl": "https://github.com/org/focus-timer-landing",
    "lastDeployedAt": "2026-06-27T14:30:00Z"
  }
}
```

`landingPageUrl` is the canonical public URL. `vercelDeploymentUrl` is the latest deployment URL from Vercel and may differ during preview deploys.

### After experiment decision

```json
{
  "status": "winner"
}
```

or

```json
{
  "status": "killed"
}
```

Only automation or an approved decision node should set terminal statuses.

### Write-back safety

- Read full `app.json` before write
- Merge updates in memory
- Write atomically (upload replaces file)
- Log previous values for audit
- Never modify `appId` or `specVersion` during automation

## Webhook placeholders

`tracking.webhooks` values are `null` until provisioned. n8n can:

1. Create webhook URLs (n8n Webhook nodes)
2. Write URLs into `app.json` before `status: ready`
3. Fire webhooks at stage completion

| Webhook key | When to fire |
|-------------|--------------|
| `validationComplete` | Schema and file checks pass |
| `deployComplete` | Mockup and landing page URLs are live |
| `emailCaptured` | Landing page form submission |
| `buyNowClicked` | User clicks purchase CTA |

### Webhook payload (recommended shape)

```json
{
  "appId": "habit-stack",
  "event": "email_captured",
  "timestamp": "2026-06-26T12:00:00Z",
  "analytics": {
    "experimentId": "exp_habit-stack_2026q2_001"
  },
  "metadata": {}
}
```

## Status branching

Recommended Switch node on `$json.status`:

| Status | Workflow |
|--------|----------|
| `draft` | Skip — notify author if accidentally triggered |
| `ready` | Full validation → deploy → ads pipeline |
| `validating` | Metrics polling, budget checks, decision rules |
| `paused` | Skip ad changes; optional alert |
| `winner` | Notify build team; optional `appStore` prep workflow |
| `killed` | Stop ads; archive; notify author |
| `built` | No validation pipeline; optional App Store sync |

## Experiment and budget guards

Before creating ads, assert:

```javascript
const exp = $json.experiment;
if (!exp?.testBudget?.amount || !exp?.hypothesis) {
  throw new Error('experiment incomplete — cannot launch ads');
}

const platforms = $json.ads?.platforms;
if (!platforms?.length) {
  throw new Error('ads.platforms required before launching paid social');
}
```

During `validating`, compare spend against `experiment.testBudget.amount` and elapsed days against `durationDays`. Pause or kill per `decisionRules`.

## UTM template expansion

`ads.utmTemplate` may be a **structured object** (preferred since 1.1.0) or a **legacy query string**. Expand before ad creation.

### Structured object (preferred)

```javascript
const tpl = $json.ads.utmTemplate;
const params = new URLSearchParams();

if (typeof tpl === 'object' && tpl !== null) {
  for (const [key, value] of Object.entries(tpl)) {
    if (value) {
      const utmKey = key === 'source' ? 'utm_source'
        : key === 'medium' ? 'utm_medium'
        : key === 'campaign' ? 'utm_campaign'
        : key === 'content' ? 'utm_content'
        : key === 'term' ? 'utm_term'
        : key;
      params.set(utmKey, String(value)
        .replace('{{appId}}', $json.appId)
        .replace('{{headline_variant}}', 'a'));
    }
  }
} else if (typeof tpl === 'string') {
  // Legacy string form — replace placeholders then parse
  const expanded = tpl
    .replace(/\{\{appId\}\}/g, $json.appId)
    .replace(/\{\{headline_variant\}\}/g, 'a');
  expanded.split('&').forEach(pair => {
    const [k, v] = pair.split('=');
    if (k && v) params.set(k, v);
  });
}

const destination = `${$json.deployment.landingPageUrl}?${params.toString()}`;
```

### Legacy string only

```javascript
const template = $json.ads.utmTemplate;
const utm = template
  .replace('{{appId}}', $json.appId)
  .replace('{{headline_variant}}', 'a');
const destination = `${$json.deployment.landingPageUrl}?${utm}`;
```

Use the structured form in new packages. The string form remains valid for 1.0.x packages.

## Error handling

| Error | Action |
|-------|--------|
| `app.json` not found | Log, skip folder, alert if folder name looks like an app |
| Schema validation failure | Do not deploy; notify with field paths |
| Missing referenced file | Block pipeline; list missing paths |
| Deploy command failure | Set `status: paused`; fire alert webhook |
| Ad platform API error | Retry with backoff; do not change `status` |

## specVersion handling

Read `specVersion` before validation:

```javascript
const version = $json.specVersion;
// Phase 2: select schema file for version
// 1.0.x and 1.1.x → schemas/app.schema.json (single schema, backward compatible)
// 2.x.x → future split schema when breaking changes ship
```

Packages at `1.0.0` remain valid against the current schema if they omit 1.1.0 optional fields. Workflows should not reject `1.0.0` packages solely for using an older `specVersion` unless a validator confirms incompatibility.

If `specVersion` is unsupported (e.g. `2.0.0` before a schema exists), block the pipeline and notify — do not silently validate against the wrong schema.

## Testing without Drive

Use local example packages for workflow development:

- [examples/minimal-app/app.json](../examples/minimal-app/app.json) — schema-only tests
- [examples/full-app/app.json](../examples/full-app/app.json) — full pipeline dry runs

Replace Drive read nodes with **Read Binary File** pointed at local paths during development.

## Related documents

- [workflow.md](workflow.md) — pipeline stages
- [naming-conventions.md](naming-conventions.md) — folder and file rules
- [APP_PACKAGE_SPEC.md](../APP_PACKAGE_SPEC.md) — field reference
