---
name: share-kb
description: Post investigation findings or best practices to Rundeck TAM Knowledge Base in Confluence
user_invocable: true
---

# Share to TAM Knowledge Base Skill

Posts content from local investigations or best practices to the Rundeck TAM Knowledge Base in Confluence for team collaboration.

## Usage

```
/share-kb <title> [source-file-path]
```

**Examples:**
- `/share-kb "Log4j2 Execution Separation" investigations/2026-04-01-lseg-logging-cloudwatch-separation.md`
- `/share-kb "Webhook Timeout Best Practices" best-practices/webhooks-timeout-handling.md`
- `/share-kb "LSEG Investigation Summary"` (will prompt for content)

## What This Skill Does

1. **Read content** - From specified file or prompt user for content
2. **Format for Confluence** - Convert markdown to Confluence-compatible format
3. **Post to TAM KB** - Create page under Rundeck TAM Knowledge Base
4. **Return URL** - Provide direct link to published page

## Instructions

When this skill is invoked, follow these steps:

### Step 1: Parse Arguments

Extract from arguments:
- `title`: The page title (required)
- `source_file`: Optional path to local markdown file

**If source file not provided:**
- Check if there's an investigation or best practice that needs sharing
- Prompt user: "Would you like me to create content based on a recent investigation? Or provide content directly?"

### Step 2: Read Source Content

**If source file provided:**
```bash
Read <source_file>
```

**Parse and extract:**
- Title (use H1 or filename if title arg not provided)
- Problem statement
- Solution overview
- Key findings
- Technical details
- Code snippets or configurations
- Related resources

**If no source file:**
- Ask user to provide content or point to recent work in conversation

### Step 3: Format Content for Confluence

**Structure the content with these sections:**

```markdown
## Problem
[Concise problem statement]

## Root Cause
[What causes the issue - if applicable]

## Solution Overview
[High-level solution approach]

## Key Discovery
[Critical insights or non-obvious findings]

## Technical Details
[Implementation specifics, configuration, commands]

## Validation/Testing
[How to verify the solution works]

## Configuration/Code
[Relevant configuration snippets or code blocks]

## Key Takeaways
[Numbered list of important lessons]

## Customer Use Case
[Anonymized customer context - remove identifying info]

## Related Resources
[Links to documentation, GitHub issues, Jira tickets]
```

**Important formatting rules:**
- Remove customer-identifying information (company names, contacts, domains, account IDs)
- Replace with generic terms: "enterprise customer", "financial services company", "large-scale deployment"
- Keep technical details but sanitize customer-specific values
- Use code blocks for configurations and commands
- Include validation steps or test results if available

### Step 4: Post to Confluence

**Confluence details:**
- **Cloud ID:** `pagerduty.atlassian.net`
- **Space ID:** `2259124765` (RUNDECK space)
- **Parent Page ID:** `3322773518` (Rundeck TAM - Knowledge Base)
- **Content Format:** `markdown`

Post through the Runlayer connector. Call `search_tools` to locate the Confluence page-creation tool, then `execute_tool` with the exact tool_name and arguments returned. Do not call `mcp__atlassian__createConfluencePage` directly. Pass parameters equivalent to:

```bash
cloudId: "pagerduty.atlassian.net"
spaceId: "2259124765"
parentId: "3322773518"
title: "<page-title>"
contentFormat: "markdown"
body: "<formatted-content>"
```

### Step 5: Report Success

After successful post, provide:

```markdown
✅ Posted to TAM Knowledge Base!

**Page:** <title>
**URL:** <confluence-url>
**Space:** Rundeck TAM - Knowledge Base

Next steps:
- Review page formatting in Confluence
- Add attachments if needed (configuration files, screenshots)
- Share URL with team in #rundeck-tams Slack channel
```

## Content Quality Guidelines

**Focus on reusability:**
- Make content applicable beyond single customer
- Highlight patterns and common scenarios
- Document discovery techniques and troubleshooting approaches

**Be concise but complete:**
- Problem statement in 1-2 sentences
- Solution overview in 1 paragraph
- Technical details can be comprehensive
- Use tables for validation results or comparison data

**Include validation:**
- Test results showing solution works
- Commands to reproduce or verify
- Expected vs actual behavior

**Anonymize customer data:**
- Remove: company names, contacts, account IDs, domains, ticket numbers
- Keep: technical details, error messages, version numbers, architecture patterns

## Error Handling

**If file not found:**
- Verify path is correct
- Suggest searching: `Glob pattern="*<keyword>*" path="investigations/"` or `path="best-practices/"`
- Offer to list recent investigations: `ls -lt investigations/*.md | head -10`

**If Confluence API fails:**
- Check API connection
- Verify space ID and parent page ID are correct
- Suggest manual post: "You can manually create page at https://pagerduty.atlassian.net/wiki/spaces/RUNDECK/pages"

**If content formatting issues:**
- Confluence markdown has limitations (no nested tables, some syntax differences)
- Suggest: "Review page after posting and adjust formatting in Confluence editor if needed"

**If title conflicts:**
- Confluence may reject duplicate titles
- Suggest adding date suffix: "Title (2026-04-14)" or versioning: "Title v2"

## Use Cases

**After investigation resolution:**
- Post findings to share solution with team
- Makes knowledge searchable for future similar cases

**Pattern documentation:**
- Multiple customers hit same issue
- Create TAM KB page to centralize solution

**Best practice sharing:**
- Discovered configuration or approach that works well
- Document for team reference

**Configuration examples:**
- Complex configurations that work (log4j2, plugins, integrations)
- Provide copy-paste examples for team

## Notes

- Average execution time: 10-20 seconds
- Page created under "Rundeck TAM - Knowledge Base" parent page
- Markdown support in Confluence has some limitations vs GitHub-flavored markdown
- Complex tables or nested formatting may need manual adjustment in Confluence
- Attachments (log files, config files) must be uploaded manually after page creation
- Consider adding page labels/tags in Confluence after creation for better discoverability
