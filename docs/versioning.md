# Versioning

How the App Package Specification is versioned, how `specVersion` works in `app.json`, and what to expect when the spec changes.

## Two version numbers

| Version | Where | Purpose |
|---------|-------|---------|
| **Spec version** | `app.json` → `specVersion` | Pins the contract for that package |
| **Repo release** | Git tags, [CHANGELOG.md](../CHANGELOG.md) | Tracks spec + docs + schema releases |

They move together on releases but serve different consumers:

- Validators read `specVersion` from each package
- Maintainers read CHANGELOG and git tags for repo history

## specVersion format

- [Semantic Versioning](https://semver.org/): `MAJOR.MINOR.PATCH`
- Pattern in schema: `^\d+\.\d+\.\d+$`
- No pre-release or build metadata in `app.json`

Current spec version: **1.0.0**

## Semver rules for this spec

### MAJOR (e.g. 1.0.0 → 2.0.0)

Increment when making **breaking** changes:

- Removing a field or section
- Renaming a field
- Changing a field's type
- Tightening validation in a way that invalidates previously valid packages
- Changing enum values (removing or redefining)

Packages with the old `specVersion` must be migrated before validation passes.

### MINOR (e.g. 1.0.0 → 1.1.0)

Increment when adding **backward-compatible** features:

- New optional top-level section
- New optional field in an existing section
- New enum value that existing packages can ignore

Existing packages with the previous minor version remain valid if they bump `specVersion`.

### PATCH (e.g. 1.0.0 → 1.0.1)

Increment for **non-breaking clarifications**:

- Documentation fixes
- JSON Schema description changes
- Stricter schema bugs that restore intended behavior

Packages typically bump `specVersion` on patch releases for consistency, but unchanged packages may still validate if the schema change is documentation-only.

## What packages must declare

Every `app.json` includes:

```json
{
  "specVersion": "1.0.0"
}
```

Validators (Phase 2) will:

1. Read `specVersion`
2. Select the matching schema from `schemas/`
3. Reject packages with missing or unsupported versions

## Migration guidance

When a new spec version ships:

1. Read [CHANGELOG.md](../CHANGELOG.md) for what changed
2. Update `specVersion` in your `app.json`
3. Add new fields if the minor version introduced them
4. Rename or restructure fields if the major version changed them
5. Re-validate against the new schema

### Example: hypothetical 1.1.0

If 1.1.0 adds an optional `notifications` section:

```json
{
  "specVersion": "1.1.0",
  "notifications": {
    "pushEnabled": false
  }
}
```

Packages without `notifications` remain valid after bumping `specVersion` to `1.1.0`.

### Example: hypothetical 2.0.0

If 2.0.0 renames `commerce.cta.buyNowText` to `commerce.cta.purchaseText`:

- All packages must update the field name
- Automated migration script (Phase 2+) recommended
- Packages on `1.x` fail validation until migrated

## Repo release process

Recommended maintainer workflow:

1. Update `schemas/app.schema.json`
2. Update [APP_PACKAGE_SPEC.md](../APP_PACKAGE_SPEC.md)
3. Update [templates/app.json](../templates/app.json) and [examples/](../examples/)
4. Add entry to [CHANGELOG.md](../CHANGELOG.md)
5. Tag release: `v1.0.0` (repo) matching spec `1.0.0`

## Validator version selection (Phase 2)

Planned behavior:

```txt
specVersion 1.0.x → schemas/app.schema.json
specVersion 1.1.x → schemas/app.schema.json (or schemas/app.schema.v1.1.json if split)
specVersion 2.x.x → schemas/app.schema.v2.json
```

For 1.0.0, a single schema file is sufficient. Split schemas when a major version requires supporting multiple versions simultaneously.

## n8n and tooling

- n8n workflows should read `$json.specVersion` before validation nodes
- AI generators should be given the current `specVersion` in prompts
- Dashboards can group experiments by `specVersion` to compare cohorts fairly

## Deprecation policy

When deprecating a field:

1. **Minor release:** Mark deprecated in APP_PACKAGE_SPEC.md; field still validates
2. **Next major release:** Remove field from schema
3. Provide migration notes in CHANGELOG for at least one major cycle

## Current status

| Item | Version |
|------|---------|
| Spec | 1.0.0 |
| Schema | [schemas/app.schema.json](../schemas/app.schema.json) |
| Released | 2026-06-26 |

## Related documents

- [CHANGELOG.md](../CHANGELOG.md) — release history
- [APP_PACKAGE_SPEC.md](../APP_PACKAGE_SPEC.md) — normative contract
- [design-philosophy.md](design-philosophy.md) — versioning rationale
