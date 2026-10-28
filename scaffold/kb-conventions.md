<!-- TAM-KB:BEGIN — managed by automation-tam-kb-skills setup. Safe to edit; keep the markers so setup can update this block. -->
# TAM Knowledge Base — Conventions

This project is a Technical Account Manager knowledge base for PagerDuty Rundeck
(Runbook Automation) customer support. It organizes customer investigations, best
practices, demos, and solutions for faster case resolution. These conventions are
consumed by the `automation-tam-kb-skills` plugin (`/investigate`, `/best-practice`,
`/share-kb`, plus the Rundeck job generator and Mermaid diagrams).

## Directory structure

- `customers/` — one profile per customer (`company-name.md`)
- `investigations/` — one file per case (`YYYY-MM-DD-brief-kebab-description.md`)
- `best-practices/` — one guide per topic (`topic-area-kebab-description.md`)
- `demo/` — self-contained demo packages
- `artifacts/genai-jobs/` — generated Rundeck job YAML
- `repositories/` — optional local checkouts of reference repos (docs, source, job definitions)
- Each content directory has an `INDEX.md`; `*/TEMPLATE.md` defines file structure.

## Naming

Lowercase, hyphenated. Customers `company-name.md`; investigations
`YYYY-MM-DD-brief-kebab-description.md`; best practices `topic-kebab.md`.

## Configuration

Per-user settings live in `.tam-kb/config.yml` (written by `/tam-kb-setup`): Confluence
space, Jira project, GitHub repos, connector mode, and KB paths. Skills read config
rather than hardcoding these values. The presence of `.tam-kb/config.yml` marks the KB
root. If it is missing, run `/tam-kb-setup` first.

## Connectors

External systems (Jira, Confluence, GitHub, PagerDuty) are reached through connectors.
When `connectors.mode` is `runlayer` (default in Cowork), discover the tool with
`search_tools` then call `execute_tool` — do not assume `mcp__atlassian__*` tool names
and do not shell out to the `gh` CLI. When `connectors.mode` is `direct-mcp`, the direct
MCP tools may be used instead.

## Workflow priority

Search order for answering a question: knowledge base (`investigations/`,
`best-practices/`, `customers/`) → generated/known job definitions → official docs →
product source (if `repositories/` present) → GitHub issues / Jira. Always cite sources
and file paths. Update the relevant `INDEX.md` when adding a document.

## Confidentiality

Never post customer-identifying information (names, contacts, account IDs, URLs, ticket
numbers) to public GitHub. Anonymize before sharing externally (e.g. "enterprise
financial-services customer"). Keep technical detail; sanitize customer-specific values.
`/share-kb` anonymizes before posting to Confluence — keep that gate intact.

## Advisory posture

Position PagerDuty solutions for business value (MTTR, cost, compliance) and
orchestration, not just point fixes. See `.claude/memory/tam-advisory-philosophy.md`.
<!-- TAM-KB:END -->
