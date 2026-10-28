---
name: best-practice
description: Automate best practice documentation workflow - research knowledge base, docs, code, community, synthesize findings, generate document, update INDEX
user_invocable: true
---

# Best Practice Documentation Skill

Automates the complete workflow for documenting best practices in the TAM knowledge base, reducing 2-4 hours of manual work to 10-15 minutes.

## Usage

```
/best-practice <topic-name>
/bp <topic-name>
```

**Examples:**
- `/bp webhook-timeout-patterns`
- `/best-practice runner-disk-management`
- `/bp ansible-integration-patterns`

## What This Skill Does

Executes the complete best practice documentation workflow:
1. **Validate & check existing** - Ensure no duplicate, handle updates
2. **Research knowledge base** - Find similar investigations, best practices, customer examples
3. **Search documentation** - Find official guidance and version information
4. **Search product code** - Verify technical details and defaults (when relevant)
5. **Search GitHub** - Find community issues, bugs, and feature requests
6. **Search Jira** - Locate internal development tickets and fix versions
7. **Community search** - WebFetch external sources if needed (optional)
8. **Synthesize findings** - Present research summary for user confirmation
9. **Generate document** - Create best practice following TEMPLATE.md structure
10. **Update INDEX.md** - Add to categories, alphabetical table, and quick reference
11. **Report completion** - Comprehensive summary with next steps

## Instructions

When this skill is invoked, follow these steps:

### Step 1: Parse Arguments & Validate

Extract the topic name from arguments:
- Required: topic name (e.g., "webhook-timeout-patterns")
- Convert to kebab-case if not already
- If topic is >80 chars, warn: "Topic seems broad. Consider splitting?"

If no topic provided:
- Error: "Usage: /best-practice <topic-name>"
- Show examples

### Step 2: Check for Existing Best Practice

Search for existing best practices with similar names:
```bash
Glob pattern="*${keywords}*" path="best-practices/"
```

**Decision logic:**
- **Exact match:** Ask user: "Best practice exists at `<file>`. (1) Update existing, (2) Create new with different focus, (3) Cancel?"
- **Partial matches:** List them, ask: "Found similar: `<files>`. (1) Update one, (2) Create new, (3) Cancel?"
- **No matches:** Proceed with creation

If user chooses update, note the filename for later Edit instead of Write.

### Step 3: Search Knowledge Base

Extract search keywords from topic name (remove hyphens, split into terms).

**Search best practices:**
```bash
Grep pattern="<keywords>" path="best-practices/" output_mode="content" -C 3 head_limit=50
```

**Search investigations:**
```bash
Grep pattern="<keywords>" path="investigations/" output_mode="content" -C 3 head_limit=50
```

**Search customer profiles (if relevant):**
```bash
Grep pattern="<keywords>" path="customers/" output_mode="files_with_matches"
```

If customer profiles found, Read them to gather context.

**Collect and note:**
- Related best practices (for "Related Best Practices" section)
- Investigation examples (for "Real-World Examples" section)
- Common tags from similar documents (for metadata)
- Customer context when applicable

### Step 4: Search Official Documentation

```bash
Grep pattern="<keywords>" path="repositories/docs/" output_mode="content" -C 2 head_limit=30
```

**Collect and note:**
- Official guidance and recommendations
- Version-specific information
- Configuration references and options
- Known limitations or caveats

### Step 5: Search Product Code (Conditional)

**When to search:**
- Topic involves specific feature behavior
- Need default configuration values
- Understanding performance characteristics
- Verifying technical constraints

```bash
Grep pattern="<component|class>" path="repositories/rundeckpro/" output_mode="content" head_limit=30
```

**Collect and note:**
- Default configuration values
- Implementation details and behavior
- Technical constraints or limits
- Version-specific logic

### Step 6: Search GitHub Issues (via Runlayer connector)

Query GitHub through the Runlayer connector — the `gh` CLI is not available in this environment. Call `search_tools` to locate the GitHub issue-search tool, then `execute_tool` to search `rundeck/rundeck` and `rundeckpro/rundeckpro` for `<keywords>` across open and closed issues. Do not shell out to `gh`.

**Collect and note:**
- Known issues or bugs
- Feature requests and community feedback
- Workarounds and community solutions
- Issue numbers for citation

If the GitHub connector is unavailable, note it and continue.

### Step 7: Search Jira RUN Project (via Runlayer connector)

Query Jira through the Runlayer connector. Call `search_tools` to locate the Jira issue-search tool, then `execute_tool` with the exact tool_name and arguments returned. Do not call `mcp__atlassian__*` tools directly. Use parameters equivalent to:

```bash
jql: "project=RUN AND (summary ~ \"<keywords>\" OR description ~ \"<keywords>\") ORDER BY updated DESC"
maxResults: 15
```

**Collect and note:**
- Internal development tickets
- Fix versions and release information
- Development status and context
- Jira keys for citation (RUN-XXXX)

**Error handling:** If the Jira connector is unavailable, note "Jira search unavailable" and continue.

### Step 8: Community Search (Optional/As-Needed)

**Trigger when:**
- Fewer than 3 relevant sources found in Steps 3-7
- Novel or emerging pattern not well documented
- User indicates broader research needed

```bash
WebFetch with parameters:
  url: "https://groups.google.com/g/rundeck-discuss/search?q=<keywords>"
  prompt: "Find discussions about <topic>. Extract: problems reported, solutions proposed, patterns recommended, common issues"
```

**Collect and note:**
- Community solutions and patterns
- Common pitfalls and issues
- Implementation variations
- **Important:** Flag unverified claims, verify against official docs

### Step 9: Synthesize Findings & Get User Confirmation

Analyze all research findings:
1. **Identify patterns** across sources
2. **Determine recommendation:** Primary approach with justification
3. **Identify alternatives:** Other approaches and why primary is preferred
4. **Extract implementation:** Step-by-step with code examples
5. **Document benefits and trade-offs**
6. **Select metadata:**
   - **Product Area:** Determine from these categories:
     - Job Design & Orchestration
     - Node Management & Targeting
     - Security & Access Control
     - Performance & Scaling
     - Integrations & Webhooks
     - Architecture & Deployment
     - Documentation & Maintenance
   - **Applies to Version:** From Jira/docs (be specific like "Rundeck 5.6.0+" or conservative "All versions")
   - **Tags:** Use existing taxonomy from similar BPs (scan tags found in Step 3), 3-8 tags maximum

**Present research summary to user:**

```markdown
# Research Summary: <topic-name>

## Sources Found
- Knowledge Base: X investigations, Y best practices
- Documentation: N pages
- GitHub: P issues
- Jira: Q tickets
- Community: [if searched]

## Recommended Approach
[1-2 paragraph summary of primary recommendation based on research]

## Key Patterns Identified
- [Pattern 1 from research]
- [Pattern 2 from research]
- [Pattern 3 from research]

## Proposed Metadata
- **Product Area:** [determined category]
- **Applies to Version:** [version from research]
- **Tags:** #tag1 #tag2 #tag3 [from similar BPs]

## Quality Check
- ✓ [X] sources found (minimum 4 recommended)
- ✓ [Y] code/config examples available
- ✓ [Z] real-world examples found

Ready to generate document?
```

**Wait for user confirmation.** User can:
- Confirm and proceed
- Request changes to metadata or approach
- Ask for additional research
- Cancel the workflow

Do NOT proceed to generation without user confirmation.

### Step 10: Generate Best Practice Document

Once user confirms, generate the document:

1. **Read template for reference:**
```bash
Read file_path="best-practices/TEMPLATE.md"
```

2. **Populate all sections with researched content:**

**Header (lines 1-6):**
```markdown
# [Title Case Topic Name]

**Last Updated:** 2026-04-20
**Product Area:** [from Step 9]
**Applies to Version:** [from Step 9]
**Tags:** #tag1 #tag2 #tag3 [from Step 9]

---
```

**Use Case / Scenario:**
- Describe when someone needs this (from investigations + docs)
- What problem does it solve?
- What scenarios is this relevant for?

**Recommendation:**
- Clear, actionable guidance (the "what to do")
- Primary approach from synthesis
- Can include a table if multiple options

**Implementation:**
- Step-by-step how-to
- Multiple patterns if applicable (Pattern 1, Pattern 2, etc.)
- Code examples from docs and code research

```yaml
# Include actual YAML/config examples
# Use code snippets found in research
```

**Why This Works:**
- Technical explanation from code + docs research
- Underlying principles or mechanisms
- Why this approach is effective

**Benefits:**
- Numbered list from research findings
- Concrete advantages (performance, security, maintainability, etc.)
- Quantify when possible

**Trade-offs / Caveats:**
- Limitations identified in research
- Performance considerations
- Situations where this might not apply
- ⚠️ Flag conflicting information if found

**Alternatives Considered:**
- Other approaches found in research
- Why the recommended approach is preferred
- When alternatives might be better

**Real-World Examples:**
- Links to investigations: `[Customer/Case](../investigations/YYYY-MM-DD-description.md)`
- If none: "No investigations documented yet. This best practice is based on [official docs/code analysis/community patterns]."

**Related Best Practices:**
- Links to related docs from KB search
- Format: `[Title](./filename.md) - Brief description`

**Official Documentation:**
- Links to docs repo files
- GitHub issues by number
- Jira tickets by key
- Official website links

3. **Quality checks before writing:**
- ✓ All 4 metadata fields populated
- ✓ 3-8 tags from existing taxonomy
- ✓ All template sections present
- ✓ At least 1 code/config example
- ✓ At least 2 sources cited
- ✓ Proper markdown formatting

4. **Write the document:**

If creating new:
```bash
Write file_path="best-practices/<kebab-topic-name>.md" content="<generated-content>"
```

If updating existing (from Step 2):
```bash
Edit file_path="best-practices/<existing-filename>.md"
```
Update relevant sections while preserving structure.

### Step 11: Update INDEX.md

1. **Read current INDEX:**
```bash
Read file_path="best-practices/INDEX.md"
```

2. **Determine primary category** from these options (use emoji exactly as shown):
   - 🎯 Job Design & Orchestration
   - 🖥️ Node Management & Targeting
   - 🔐 Security & Access Control
   - 📊 Performance & Scaling
   - 🔗 Integrations & Webhooks
   - 🏗️ Architecture & Deployment
   - 📝 Documentation & Maintenance

3. **Update three sections using Edit tool:**

**a) Categories section:**
Find the appropriate category (e.g., "### 🎯 Job Design & Orchestration") and add entry:
```markdown
- [Title](./filename.md) - One-line description #tag1 #tag2 #tag3
```
Insert alphabetically within the category.

**b) All Best Practices (Alphabetical) table:**
Find correct alphabetical position and insert:
```markdown
| [Title](./filename.md) | Category [& Secondary Categories if applicable] | 2026-04-20 | [Related cases or "N/A"] |
```

**c) Quick Reference by Use Case section:**
Add 2-4 "I need to..." entries that reflect common use cases:
```markdown
- **...specific problem/task** → [Title](./filename.md)
```
Keep alphabetically sorted.

4. **Update "Last Updated" date:**
Change the date at the top:
```markdown
**Last Updated:** 2026-04-20
```

5. **Execute updates with Edit tool:**
Use Edit to make each change (or combine if they're close together):
```bash
Edit file_path="best-practices/INDEX.md" old_string="<existing-text>" new_string="<updated-text>"
```

### Step 12: Report Completion

Provide comprehensive completion summary:

```markdown
✅ Best Practice Created Successfully!

## Document Details
**Title:** [Full Title]
**File:** best-practices/<filename>.md
**Category:** [Primary Category]
**Tags:** #tag1 #tag2 #tag3

## Research Summary
- Knowledge Base: X investigations, Y best practices
- Documentation: N pages
- GitHub: P issues
- Jira: Q tickets
- Community: [if searched]
- **Total sources:** [count]

## Document Sections Completed
✓ Use Case / Scenario
✓ Recommendation with [count] patterns
✓ Implementation with [count] code examples
✓ Why This Works
✓ [count] Benefits listed
✓ [count] Trade-offs documented
✓ [count] Alternatives considered
✓ Real-world examples: [links or "pending documentation"]
✓ [count] Related best practices linked
✓ [count] Official documentation references

## INDEX.md Updated
✓ Added to [Category Name]
✓ Added to alphabetical table (row [N])
✓ Added [count] quick reference entries
✓ Updated "Last Updated" date

## Next Steps
1. Review the document at: best-practices/<filename>.md
2. Test implementation in sandbox if applicable
3. Share with team: `/share-kb "<title>" best-practices/<filename>.md`
4. Link from relevant customer profiles if applicable
```

If this was an update instead of creation:
```markdown
✅ Best Practice Updated Successfully!

## Updated Document
**Title:** [Full Title]
**File:** best-practices/<filename>.md
[rest of summary adapted for update scenario]
```

## Quality Guidelines

### Research Quality
- Search minimum 4 source categories (KB, docs, code, external trackers)
- Cite specific file paths for code/doc references
- Link to GitHub issues by number (#1234)
- Link to Jira tickets by key (RUN-1234)
- Verify community claims against official sources
- Flag unverified or conflicting information

### Document Quality
- **Clarity:** Each section immediately actionable or informative
- **Completeness:** All template sections filled (no TODOs)
- **Evidence-based:** All recommendations backed by sources
- **Practical:** Include real code/config examples, not just prose
- **Honest:** Trade-offs and caveats documented transparently
- **Connected:** Links to related documents and investigations

### Metadata Quality
- Product Area matches INDEX.md categories exactly
- Version info specific ("Rundeck 5.6.0+") or conservative ("All versions")
- Tags follow existing taxonomy (check similar BPs first)
- 3-8 tags that accurately categorize content
- No duplicate or overlapping tags

## Error Handling

### No Topic Provided
```
❌ Error: Topic name required
Usage: /best-practice <topic-name>
Examples:
  /bp webhook-timeout-patterns
  /best-practice runner-disk-management
```

### Topic Too Broad
```
⚠️ Topic seems broad (>80 chars or covers multiple areas).
Consider splitting into subtopics:
  - [specific subtopic 1]
  - [specific subtopic 2]
  
Continue anyway? (yes/no)
```

### Insufficient Research (<3 sources)
```
⚠️ Limited research results found (<3 sources).

Sources found:
- [list what was found]

Options:
1. Proceed with available data (document will note limitations)
2. Conduct manual research first, then retry
3. Create investigation first to document real-world case
4. Cancel and revisit later

Choose option (1-4):
```

### Existing Best Practice Found
```
ℹ️ Found existing best practice(s):
  - best-practices/<filename>.md - [brief description]

Options:
1. Update existing document
2. Create new with different focus
3. Cancel

Choose option (1-3):
```

### External Tracker Failures
- Note in synthesis summary: "GitHub search unavailable" or "Jira search failed"
- Continue with available sources
- Flag in completion: "Note: [source] was unavailable, recommend manual verification"

### Conflicting Information
- Document both sources in "Trade-offs / Caveats" with ⚠️ callout
- Present context for each source
- Recommend sandbox testing for verification
- Flag in completion report

### No Real-World Examples
- Use: "No investigations documented yet. This best practice is based on official documentation and code analysis."
- Still create the BP if research is otherwise solid
- Note in completion: "Suggest documenting future implementations"

### No Customer Profile Found
- Continue without customer context
- Still search investigations for customer name mentions
- Note in completion if recurring customer

## Notes

- **Average execution time:** 10-15 minutes (2-3 min research, 5-7 min generation, 2-3 min INDEX update, plus user review time)
- **When to use:** Documenting recurring patterns, consolidating scattered knowledge, establishing team standards
- **When NOT to use:** One-off solutions (use investigation instead), unclear patterns (gather more data first)
- **Integration:** Can follow `/investigate` workflow, precedes `/share-kb` for team sharing
- **Related workflows:** See CLAUDE.md "When Helping Me..." section for complete TAM workflows