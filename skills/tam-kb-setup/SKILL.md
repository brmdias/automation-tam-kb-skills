---
name: tam-kb-setup
description: Bootstrap or update a TAM knowledge base workspace for the automation-tam-kb-skills plugin. Use when a user installs the plugin and has no KB structure yet, when `.tam-kb/config.yml` is missing, when another TAM skill (investigate, best-practice, share-kb) reports it cannot find the KB root, or when the user says "set up the TAM KB", "initialize the knowledge base", "onboard me to the TAM plugin", or wants to update the scaffold/config after a plugin upgrade.
user_invocable: true
---

# TAM KB Setup

Provision (or update) a Technical Account Manager knowledge base in the user's project so
the other skills in this plugin have the structure, templates, and configuration they
expect. Idempotent: safe to re-run — it fills gaps and updates managed blocks without
overwriting the user's own content.

## Your role

Guide the user through a short, mostly-automatic setup. Keep prompts minimal and provide
sensible defaults. Never overwrite customer data, investigations, or best practices the
user has already written.

## Inputs (bundled scaffold)

All seed content ships inside this plugin at `${CLAUDE_PLUGIN_ROOT}/scaffold/`:

```
scaffold/
├── customers/INDEX.md
├── investigations/INDEX.md
├── investigations/TEMPLATE.md
├── best-practices/INDEX.md
├── best-practices/TEMPLATE.md
├── artifacts/genai-jobs/.gitkeep
├── demo/.gitkeep
├── claude-memory/tam-advisory-philosophy.md          → target .claude/memory/
├── claude-memory/sandbox-environments.template.md    → rendered to target .claude/memory/sandbox-environments.md
└── kb-conventions.md                                 → merged into target CLAUDE.md
```

Never read seed files from the user's repo — always copy from `${CLAUDE_PLUGIN_ROOT}/scaffold/`.

## Workflow

### 1. Resolve the target directory

Default to the current working folder. Confirm with the user. Determine mode:

- If `.tam-kb/config.yml` does **not** exist → **fresh setup**.
- If it exists → **update mode**: keep all existing content and config values; only add
  missing scaffold files and refresh managed blocks. Tell the user you're updating, not
  recreating.

Never delete or overwrite existing `customers/`, `investigations/`, or `best-practices/`
content.

### 2. Copy the scaffold (only missing files)

For each path under `${CLAUDE_PLUGIN_ROOT}/scaffold/` (except `claude-memory/` and
`kb-conventions.md`, handled below), create it in the target **only if it does not already
exist**. This creates `customers/`, `investigations/`, `best-practices/`, `artifacts/genai-jobs/`,
and `demo/` with their `INDEX.md` and `TEMPLATE.md` seeds. Preserve any file the user already has.

### 3. Place the memory files

- Copy `scaffold/claude-memory/tam-advisory-philosophy.md` → `<target>/.claude/memory/tam-advisory-philosophy.md` (only if missing).
- Render `scaffold/claude-memory/sandbox-environments.template.md` → `<target>/.claude/memory/sandbox-environments.md`. In fresh setup, either fill placeholders with the user's answers from step 4 or leave the `<placeholders>` for them to complete. In update mode, do not overwrite an existing `sandbox-environments.md`.

If writing under `.claude/` is blocked in the current environment, write the two files to
`<target>/claude-memory/` instead and tell the user to move them to `.claude/memory/`.

### 4. Collect config values

Prompt the user (use a single multi-question step; offer the defaults):

| Value | Config key | Default |
| --- | --- | --- |
| Confluence space key | `confluence.space_key` | `RUNDECK` |
| Confluence space ID | `confluence.space_id` | `2259124765` |
| Confluence base URL | `confluence.base_url` | `https://pagerduty.atlassian.net/wiki` |
| Jira project key | `jira.project` | `RUN` |
| GitHub repos to search | `github.repos` | `rundeck/rundeck`, `rundeckpro/rundeckpro` |
| Project name | `project.name` | folder name |

The Confluence/Jira/GitHub defaults are org-shared — accept-as-default is expected; only
override if the user targets a different space/project. In update mode, pre-fill prompts
with the existing config values.

### 5. Detect connector mode

If a Runlayer connector is available in this environment, set `connectors.mode: runlayer`;
otherwise `connectors.mode: direct-mcp`. Record it in config.

### 6. Write `.tam-kb/config.yml`

Write the marker/config file (this file marks the KB root for all other skills):

```yaml
version: 1
project:
  name: "<project name>"
  root_marker: true
confluence:
  base_url: "https://pagerduty.atlassian.net/wiki"
  space_key: "RUNDECK"
  space_id: "2259124765"
jira:
  project: "RUN"
github:
  repos:
    - "rundeck/rundeck"
    - "rundeckpro/rundeckpro"
connectors:
  mode: "runlayer"
paths:
  customers: "customers"
  investigations: "investigations"
  best_practices: "best-practices"
  genai_jobs: "artifacts/genai-jobs"
  repositories: "repositories"
```

In update mode, preserve existing values and only add keys introduced by a newer schema
(bump `version` if the schema changed).

### 7. Merge conventions into CLAUDE.md

Take `${CLAUDE_PLUGIN_ROOT}/scaffold/kb-conventions.md` (delimited by `<!-- TAM-KB:BEGIN -->`
… `<!-- TAM-KB:END -->`). If the target `CLAUDE.md` is absent, create it from that block.
If present, replace the existing `TAM-KB:BEGIN…END` block if one exists, else append the
block. Never disturb the user's own CLAUDE.md content outside the markers.

### 8. Optional — reference repositories

Ask whether to clone the large reference repos into `repositories/` (docs, rundeckpro).
This is the only step that uses `git clone` and is entirely optional; skip gracefully if
the user declines or lacks access. Skills fall back to knowledge base + connectors when
these are absent.

### 9. Verify and report

Confirm and print a concise report:

- Directories created vs. preserved
- Memory files placed (and any fallback location used)
- Config written, with connector mode
- CLAUDE.md created vs. block updated
- Reference repos: cloned / skipped

Finish by pointing the user to the next actions: `/investigate`, `/best-practice`,
`/share-kb`, and the Rundeck job generator.

## Rules

- Idempotent and non-destructive: create-if-missing, update managed blocks only.
- Read seeds only from `${CLAUDE_PLUGIN_ROOT}/scaffold/`, never from the user's repo.
- Do not hardcode absolute paths; resolve everything relative to the confirmed target root.
- Keep secrets out of any shared file; `sandbox-environments.md` references a secrets path, never an inline token.
