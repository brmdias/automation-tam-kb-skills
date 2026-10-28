---
name: investigate
description: Run full investigation workflow for customer issue - searches knowledge base, documentation, code, and external trackers, then recommends test environment
user_invocable: true
---

# Investigation Workflow Skill

Automates the complete TAM investigation process for customer issues.

## Usage

```
/investigate <customer-name> <issue-description>
```

**Examples:**
- `/investigate LSEG "webhook timeout error"`
- `/investigate "Network International" "BeyondTrust plugin authentication"`
- `/investigate Roche "scheduler not triggering jobs"`

## What This Skill Does

Executes the full investigation workflow from CLAUDE.md:
1. **Find customer profile** - Locate and read customer environment details
2. **Search knowledge base** - Find similar investigations and best practices
3. **Check documentation** - Verify if behavior is documented
4. **Verify in code** - Search product source for actual behavior
5. **Check external trackers** - Search GitHub issues and Jira for known bugs/development
6. **Recommend test environment** - Suggest appropriate sandbox based on issue type
7. **Summarize findings** - Provide structured summary with references
8. **Document investigation** - Create new investigation file in investigations/ using template

## Instructions

When this skill is invoked, follow these steps:

### Step 1: Extract Parameters
From the arguments, identify:
- `customer_name`: The customer/company name
- `issue_keywords`: Key terms describing the issue (error messages, features, components)

### Step 2: Find Customer Profile
```bash
Grep pattern="<customer_name>" path="customers/" output_mode="files_with_matches"
```

If found:
- `Read` the customer profile file
- Note: environment (cloud/on-prem), Rundeck version, scale, past issues
- Store customer context for later

If not found:
- Note: "Customer profile not found - may be prospect or first engagement"
- Continue investigation

### Step 3: Search Knowledge Base

**Similar investigations:**
```bash
Grep pattern="<issue_keywords>" glob="investigations/**/*.md" output_mode="content" -C 3
```

**Related best practices:**
```bash
Grep pattern="<issue_keywords>" glob="best-practices/**/*.md" output_mode="content" -C 2
```

For each result:
- Note the file path, date (from filename if investigation), and key findings
- Identify if exact match or similar pattern
- Extract solutions or workarounds

### Step 4: Check Documentation
```bash
Grep pattern="<issue_keywords>" path="repositories/docs/" output_mode="content" -C 2
```

Determine:
- Is this documented behavior?
- Are there configuration options or known limitations?
- Does documentation match customer expectation?

### Step 5: Verify in Product Code
```bash
Grep pattern="<relevant_class_or_method>" path="repositories/rundeckpro/" output_mode="content" head_limit=50
```

Look for:
- Actual implementation behavior
- Default values or constants
- Error message sources
- Version-specific logic

### Step 6: Check External Trackers

**GitHub community issues (via Runlayer connector):**

Query GitHub through the Runlayer connector. Call `search_tools` to locate the GitHub issue-search tool, then `execute_tool` to search `rundeck/rundeck` and `rundeckpro/rundeckpro` for `<issue_keywords>` across open and closed issues (limit ~10 each). Do not shell out to `gh`.

**Jira development tracking (via Runlayer connector):**

Query Jira through the Runlayer connector. Call `search_tools` to locate the Jira issue-search tool, then `execute_tool` with the exact tool_name and arguments returned to search project `RUN` for `<issue_keywords>` ordered by updated date (equivalent JQL: `project=RUN AND text~"<issue_keywords>" ORDER BY updated DESC`). Do not call `mcp__atlassian__*` tools directly.

For each relevant result:
- Note issue number, status, and summary
- Check if fixed in specific version
- Note any workarounds mentioned

### Step 7: Recommend Test Environment

Do this only if sandbox environments are available. If not, note: "No sandbox environments available - recommend local WAR for quick reproduction".
Based on issue characteristics, recommend sandbox:

**Decision logic:**
Analyse the sandbox environments available and suggest the most appropriate one.
Reference: `Read .claude/memory/sandbox-environments.md` for connection details if needed
Do not connect to sandbox within this skill - just provide recommendation.

### Step 8: Generate Structured Summary

Provide findings in this format:

```markdown
# Investigation Summary: <customer_name> - <issue_description>

## Customer Context
- **Profile:** [found/not found]
- **Environment:** [cloud/on-prem, version, scale]
- **Past Issues:** [relevant history if any]

## Knowledge Base Findings

### Similar Investigations
- [investigation-file.md](path) - <brief summary>
- [Status: resolved/workaround/open]

### Relevant Best Practices
- [best-practice-file.md](path) - <brief summary>

## Documentation Status
- **Documented:** [Yes/No/Partial]
- **References:** [file paths with line numbers]
- **Key Details:** [relevant configuration, limitations, or guidance]

## Code Analysis
- **Component:** [class/method names]
- **Behavior:** [actual implementation behavior]
- **Source:** [file_path:line_number]

## External Tracker Status

### GitHub Issues
- rundeck/rundeck#<number> - <summary> [status]
- rundeckpro/rundeckpro#<number> - <summary> [status]

### Jira Issues
- RUN-<number> - <summary> [status, fix version if applicable]

## Recommended Test Environment
**Environment:** [AWS EKS / AWS EC2 / Local WAR / PDT-RDTAM]
**Reason:** [database type / issue specifics / testing needs]
**Connection:** See .claude/memory/sandbox-environments.md

## Next Steps
1. [Immediate action item]
2. [Testing recommendation]
3. [Customer communication guidance]

## Related Resources
- Customer profile: [path if exists]
- Investigations: [list of related files]
- Best practices: [list of applicable guides]
- Documentation: [key doc pages]
```

## Quality Guidelines

**Be thorough but concise:**
- Don't include every search result - prioritize relevant findings
- Cite file paths for all references (e.g., `investigations/2026-03-15-issue.md:45`)
- Mark unknowns clearly ("No similar cases found" vs vague statements)

**Provide actionable insights:**
- Don't just list findings - synthesize patterns
- If multiple investigations show same solution, highlight it
- If documentation conflicts with code, flag it explicitly

**Consider customer context:**
- Tailor recommendations to customer's environment (cloud vs on-prem, version)
- Reference their past issues if relevant patterns exist
- Adjust test environment recommendation to match their setup

## Error Handling

**If customer not found:**
- Continue investigation without customer context
- Note: "Recommend creating customer profile if recurring engagement"

**If no KB results:**
- Try broader search terms
- Note: "Novel issue - consider documenting as investigation after resolution"

**If external trackers unavailable:**
- Note the tool failure
- Continue with available information
- Suggest manual check: "GitHub search failed - manually check rundeck/rundeck issues"

**If ambiguous issue description:**
- Make best effort with available keywords
- Note: "Ambiguous issue description - search covered: [list terms used]"
- Suggest: "Refine search with specific error messages or component names"

### Step 9: Document the Investigation

After completing the investigation and presenting the summary, create a new investigation file to document the findings:

**File creation:**
```bash
investigations/YYYY-MM-DD-brief-kebab-case-description.md
```

**Use the template:** `Read investigations/TEMPLATE.md` for structure

**Populate with:**
- **Title:** Brief description of the issue
- **Date:** Today's date (YYYY-MM-DD format)
- **Customer:** Customer name from investigation
- **Product Area:** Relevant component (webhooks, jobs, nodes, etc.)
- **Tags:** Discovered from similar investigations, plus issue-specific tags
- **Problem Statement:** Issue description provided by user
- **Environment Details:** From customer profile (if found)
- **Investigation Steps:** Summary of search steps performed
- **Root Cause:** If identified during investigation
- **Solution/Resolution:** If found in KB or documentation
- **Related Documentation:** References discovered during search
- **Code/Documentation References:** File paths with line numbers from search results
- **Notes/Learnings:** Any patterns or insights from the investigation

**After creating the file:**
1. Update `investigations/INDEX.md` with new entry:
   - Add to appropriate category section
   - Add to chronological table
   - Add tags to "Quick Search by Tag" section
   - Update "Last Updated" date

## Notes

- Investigation creates a permanent record in the knowledge base
- Average execution time: 30-60 seconds for search + 2-3 minutes for documentation
- All findings are research - actual testing still required in recommended environment
- For strategic positioning advice, separately invoke `Read .claude/memory/tam-advisory-philosophy.md`
- Documentation ensures knowledge is captured and searchable for future investigations
