---
name: hibernate-metadata
description: >
  Query Hibernate project metadata (maintained series, branches, status)
  from hibernate.org JSON files. Use when determining which branches are
  maintained, mapping series to git branches, or checking project status.
---

# Hibernate Project Metadata

Machine-readable metadata for Hibernate projects is published at:

```
https://hibernate.org/metadata/<slug>.json
```

Known slugs: `orm`, `search`, `validator`, `reactive`, `nosql`.

## JSON schema

Top-level keys: `description`, `copyright`, `jira` (with `key`), `series`.

`series` is an object keyed by version string (e.g. `"7.4"`, `"8.0"`).
Each series entry has:

| Field | Content |
|-------|---------|
| `status` | One of: `development`, `latest-stable`, `limited-support`, `end-of-life` |
| `summary` | Human-readable feature highlights; for some projects also names compatible dependency versions (e.g. reactive's `"Release compatible with Hibernate ORM 7.4 and Vert.x 5.1"`) |
| `license` | Object with `name` (e.g. `"ASL v2"`) and `since` (version string or null) |
| `links` | Documentation URLs, Jira/GitHub issue links, categorized doc links |

There is no structured `integrations` or `version` field. Dependency
compatibility info, when present, is embedded in `summary` as free text.

## Status values

- **`development`** — the current development series (unreleased or pre-release).
- **`latest-stable`** — the current recommended stable release line.
- **`limited-support`** — older but still maintained; receives critical fixes.
- **`end-of-life`** — no longer maintained.

**Maintained** means status is not `end-of-life`.

## Mapping series to git branches

The series key (e.g. `"7.4"`) is the git branch name. Exception: the
`development`-status series sometimes uses the repo's default branch
(`main`) instead of a named branch. Check whether the branch actually
exists before assuming. Some projects (e.g. ORM) create the development
branch early; others keep development on `main` until branching.

## Example commands

List maintained series for a project:

```bash
curl -s https://hibernate.org/metadata/orm.json \
  | jq '[.series | to_entries[] | select(.value.status != "end-of-life") | {series: .key, status: .value.status}]'
```

Get the latest-stable series:

```bash
curl -s https://hibernate.org/metadata/search.json \
  | jq '.series | to_entries[] | select(.value.status == "latest-stable") | .key'
```

Extract dependency info from summaries (e.g. which ORM version reactive series target):

```bash
curl -s https://hibernate.org/metadata/reactive.json \
  | jq '[.series | to_entries[] | select(.value.status != "end-of-life") | {series: .key, summary: .value.summary}]'
```
