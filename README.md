# automation-tam-kb-skills

Claude plugin (and single-plugin marketplace) for PagerDuty Rundeck TAM knowledge-base workflows.

Skills: `investigate`, `best-practice`, `share-kb`, `tam-kb-setup`, `mermaid-diagrams`.

## Install

Claude Code:
```
/plugin marketplace add https://github.com/brmdias/automation-tam-kb-skills
/plugin install automation-tam-kb-skills@automation-tam-kb-marketplace
```
Claude Desktop (Cowork): add this repo as a marketplace in plugin settings, then install.

## Release
Bump `version` in `.claude-plugin/plugin.json`, commit, push, then update the marketplace in each client.
