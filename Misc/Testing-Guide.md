# Testing & Validation Guide

**Last Updated:** 2025-12-15 20:30 IST
**Status:** Reference Guide

#testing #validation #quality

---

## Overview

This guide provides comprehensive validation criteria and testing procedures for the SuBot project. Use this as a reference when validating documents, checking quality, or understanding what constitutes correct syntax.

**Relationship to other documents:**
- [[Project-Instructions#Post-Edit Validation]] - When and how Claude runs validation
- [[Obsidian-Guide]] - Obsidian syntax reference and examples

---

## Obsidian Syntax Validation

### Block ID Validation

**Format Rules:**
- Pattern: `^[a-z][a-z0-9-]*$` (lowercase, hyphens only)
- Structure: `^component-topic-description`
- Length: 3-50 characters
- Descriptive: Not generic (avoid `^block1`, `^section-a`, `^id`)

**Placement Rules:**
- Paragraphs: At END of paragraph text (last line)
- Headers: On separate line immediately after header
- Lists: After list ends, not mid-list
- Never inline in middle of sentences

**Uniqueness Rules:**
- No duplicate block IDs within same document
- Block IDs should be unique across project (not enforced but recommended)
- If similar content in multiple docs, differentiate: `^chassis-overview` vs `^architecture-overview`

**Test Procedure:**
```
1. Search document for all `^` patterns
2. Check each against format regex
3. Verify placement (end of paragraph/after header)
4. Check for duplicates within document
5. Verify descriptiveness (not generic)
```

**Examples:**

✅ **Correct:**
```markdown
The chassis loads plugins dynamically at runtime. ^chassis-plugin-loading

## Profile Interaction Model
^profile-interaction-model
```

❌ **Incorrect:**
```markdown
The chassis ^chassis-plugin-loading loads plugins dynamically. (inline)

## Profile Interaction Model ^profile-interaction-model (same line as header)

Important point here. ^block1 (generic name)
```

---

### Wiki Link Validation

**Format Rules:**
- Basic: `[[filename]]`
- Section: `[[filename#Section Name]]`
- Block: `[[filename#^block-id]]`
- With display: `[[filename#^block-id|display text]]`
- MUST have display text for block references

**Target Rules:**
- File must exist in project
- If section reference: Section must exist
- If block reference: Block ID must exist
- Path resolution: Relative to project root

**Test Procedure:**
```
1. Extract all wiki links: `\[\[([^\]]+)\]\]`
2. Parse into: filename, section/block, display text
3. Check file exists in project
4. If block reference: Search target file for block ID
5. If section reference: Search target file for section header
6. Verify block references have display text
```

**Examples:**

✅ **Correct:**
```markdown
[[01-architecture]]
[[01-architecture#System Architecture]]
[[01-architecture#^profile-interaction-model|profile interaction model]]
[[00-overview#^def-chassis|Chassis]]
```

❌ **Incorrect:**
```markdown
[[non-existent-file]]
[[01-architecture#^fake-block-id|text]] (block doesn't exist)
[[01-architecture#^profile-interaction-model]] (no display text)
[01-architecture] (not wiki link syntax)
```

---

### Callout Validation

**Format Rules:**
- Start: `> [!type]` or `> [!type] Title`
- Types: `warning`, `info`, `important`, `question`
- Content: Indented with `> ` prefix
- Spacing: Blank line before and after

**Usage Rules:**
- Use sparingly (not every paragraph)
- Appropriate type for content
- Concise content (2-4 lines typical)
- Specific, actionable titles

**Test Procedure:**
```
1. Find all callouts: `^> \[!(warning|info|important|question)\]`
2. Verify type is valid
3. Check content is properly indented
4. Verify spacing (blank lines)
5. Check usage density (not overused)
```

**Examples:**

✅ **Correct:**
```markdown
Some regular text.

> [!warning] Security Risk
> Storing credentials requires encryption at rest.

More regular text.
```

❌ **Incorrect:**
```markdown
> [!danger] Wrong Type (invalid type)

>[!warning] No space after >

> [!info]
Content not indented properly
```

---

### Tag Validation

**Taxonomy Rules:**
- Core: `#architecture`, `#security`, `#decision`, `#question`, `#workflow`
- Component: `#component`, `#component/chassis`, `#component/profile`
- Document: `#overview`, `#roadmap`, `#future`
- Lowercase only
- Use slashes for hierarchy: `#component/chassis`

**Placement Rules:**
- Document-level: At top after title/metadata
- Section-level: After section header
- Inline: When referencing concepts in text

**Test Procedure:**
```
1. Extract all tags: `#[a-z][a-z/-]+`
2. Verify against taxonomy
3. Check document has document-level tags
4. Verify placement appropriate
5. Check for misspellings
```

**Examples:**

✅ **Correct:**
```markdown
# Component Design

**Last Updated:** 2025-12-15

#component/chassis #architecture

### Plugin System
#component #security
```

❌ **Incorrect:**
```markdown
#Architecture (uppercase)
#component-chassis (hyphen instead of slash)
#random-tag (not in taxonomy)
```

---

## Cross-Reference Integrity

### Target Existence Testing

**What to check:**
- Every wiki link points to existing file
- Every block reference points to existing block
- No broken links
- No dead references

**Test Procedure:**
```
1. Extract all wiki links from document
2. For each link:
   a. Check file exists in project
   b. If block ref: Read target file, search for block ID
   c. If section ref: Read target file, search for section header
3. Report broken links with line numbers
```

**Common issues:**
- File renamed but links not updated
- Block ID renamed but references not updated
- File moved to different directory
- Typo in filename or block ID

---

### Bidirectional Reference Testing

**What to check:**
- If doc A links to doc B, does B link back? (should it?)
- Related documents cross-reference each other
- No orphaned documents (never referenced)

**Test Procedure:**
```
1. Build reference map (who links to whom)
2. Identify orphaned documents (no incoming links)
3. Check related documents have mutual references
4. Verify major documents referenced appropriately
```

---

### Block ID Uniqueness

**What to check:**
- No duplicate block IDs within document (critical)
- Block IDs ideally unique across project (recommended)
- Similar IDs appropriately differentiated

**Test Procedure:**
```
1. Extract all block IDs from document
2. Check for duplicates
3. Optionally: Extract block IDs from all documents
4. Check for cross-document duplicates
5. Verify similar IDs are differentiated
```

---

## Document Structure Validation

### Required Elements

**Every document must have:**
- Title (# heading at top)
- Last Updated timestamp
- Status field (Draft/Review/Approved) if applicable
- Document-level tags
- Major sections with block IDs

**Test Procedure:**
```
1. Check first line is # title
2. Search for "Last Updated:" in first 10 lines
3. Check for document-level tags in first 10 lines
4. Verify major sections (##) have block IDs
5. Check structure is logical (no skipped heading levels)
```

---

### Section Hierarchy

**Rules:**
- No skipped levels (# → ### is invalid, should be # → ## → ###)
- Logical organization (related content grouped)
- Block IDs on major sections (##, important ###)
- Related links at end of major sections

**Test Procedure:**
```
1. Extract all headers with levels
2. Check no skipped levels
3. Verify ## sections have block IDs
4. Check for Related links sections
```

---

### Metadata Consistency

**Check:**
- Timestamp format: "Last Updated: YYYY-MM-DD HH:MM IST"
- Status values: Only Draft/Review/Approved
- Tags follow taxonomy
- Consistent formatting across documents

---

## Dataview Query Validation

### Syntax Validation

**Valid elements:**
- Query type: TABLE, LIST, TASK
- FROM clause: Path or tag
- WHERE clause: Valid field comparisons
- SORT clause: Valid field + ASC/DESC
- LIMIT clause: Positive integer

**Test Procedure:**
```
1. Find all dataview blocks: ```dataview
2. Parse query structure
3. Verify query type is valid
4. Check FROM clause references valid paths/tags
5. Verify field names in WHERE/SORT are valid
6. Check syntax completeness (no incomplete clauses)
```

**Common errors:**
- Missing FROM clause
- Invalid field names
- Malformed WHERE conditions
- Syntax errors in paths

---

### Path Validation

**Check:**
- FROM paths point to existing directories/files
- Paths use correct format ("" for root, "folder" for subfolder)
- No typos in path names
- Paths accessible from project root

**Test Procedure:**
```
1. Extract FROM clause paths
2. Check each path exists in project
3. Verify path format correct
4. Test path resolves from project root
```

---

## Content Coherence Testing

### Cross-Document Consistency

**What to check:**
- Decisions in decisions-log align with architecture
- Architecture describes what roadmap plans to build
- Questions are either answered or tracked as open
- No contradictory statements across documents

**Test Procedure:**
```
1. Review decisions in 03-decisions-log.md
2. Check if architecture reflects those decisions
3. Look for contradictions in design
4. Verify questions resolved or marked pending
```

**Common issues:**
- Decision made but not reflected in architecture
- Architecture describes approach different from decision
- Old questions not marked as resolved
- Conflicting information in different docs

---

### Temporal Consistency

**What to check:**
- Timestamps are current
- Status reflects actual state
- No stale information
- Recent changes reflected across related docs

**Test Procedure:**
```
1. Check Last Updated timestamps
2. Verify timestamps match edit history
3. Look for obviously stale content
4. Check related docs updated together
```

---

## Manual Testing Procedures

### Single Document Validation

**Checklist:**
```
[ ] Title present and descriptive
[ ] Last Updated timestamp current
[ ] Document-level tags present and correct
[ ] Status field present (if applicable)
[ ] Major sections have block IDs
[ ] Block IDs at end of paragraphs/after headers
[ ] Block IDs descriptive (not generic)
[ ] No duplicate block IDs
[ ] Wiki links have display text (for block refs)
[ ] All wiki link targets exist
[ ] Callouts properly formatted
[ ] Callouts not overused
[ ] Tags follow taxonomy
[ ] Related links present in major sections
[ ] Dataview queries syntactically correct
[ ] No broken Obsidian syntax
```

**Process:**
1. Open document in Obsidian
2. Visual scan for obvious issues
3. Check each checklist item
4. Note any issues found
5. Fix or report issues

---

### Project-Wide Validation

**Checklist:**
```
[ ] All documents have required elements
[ ] No broken wiki links across project
[ ] Block IDs unique within documents
[ ] Cross-references bidirectional where appropriate
[ ] No orphaned documents
[ ] Template.md reflects current structure
[ ] Dataview queries work in Obsidian
[ ] No contradictory information
[ ] Decisions align with architecture
[ ] Questions resolved or tracked
```

**Process:**
1. Validate each document individually
2. Check cross-document references
3. Verify consistency across documents
4. Test Dataview queries
5. Report findings

---

### Quick Validation (Session End)

**Fast checks:**
```
[ ] Edited files have updated timestamps
[ ] New block IDs follow format
[ ] New wiki links have valid targets
[ ] No obvious syntax errors
[ ] Template.md updated if structure changed
```

**Process:**
1. List files edited this session
2. Quick check each file
3. Verify timestamps updated
4. Check any new references valid
5. Confirm Template.md sync

---

## Common Error Patterns

### Block ID Errors

**Pattern 1: Block ID on same line as header**
```markdown
❌ ## Section Name ^block-id
✅ ## Section Name
   ^block-id
```

**Pattern 2: Generic block IDs**
```markdown
❌ ^block1, ^section-a, ^id
✅ ^profile-interaction-model, ^chassis-plugin-loading
```

**Pattern 3: Block ID inline**
```markdown
❌ Important concept ^block-id here.
✅ Important concept here. ^block-id
```

---

### Wiki Link Errors

**Pattern 1: Missing display text for block refs**
```markdown
❌ [[01-architecture#^profile-interaction-model]]
✅ [[01-architecture#^profile-interaction-model|profile interaction model]]
```

**Pattern 2: Non-existent targets**
```markdown
❌ [[fake-file#^block]]
✅ [[01-architecture#^existing-block|text]]
```

**Pattern 3: Wrong syntax**
```markdown
❌ [01-architecture] (Markdown link, not wiki link)
✅ [[01-architecture]]
```

---

### Callout Errors

**Pattern 1: Wrong indentation**
```markdown
❌ > [!warning]
Content not indented

✅ > [!warning]
> Content properly indented
```

**Pattern 2: Invalid type**
```markdown
❌ > [!danger] (not a valid type)
✅ > [!warning]
```

**Pattern 3: Overuse**
```markdown
❌ Every paragraph in callout boxes
✅ Sparing use for truly important content
```

---

### Tag Errors

**Pattern 1: Uppercase tags**
```markdown
❌ #Architecture
✅ #architecture
```

**Pattern 2: Wrong hierarchy separator**
```markdown
❌ #component-chassis
✅ #component/chassis
```

**Pattern 3: Non-taxonomy tags**
```markdown
❌ #random-tag
✅ #architecture (from taxonomy)
```

---

## Validation Reports

### Report Format

**For each issue found:**
```
[Severity] File: Line Number - Issue Description

Example:
[CRITICAL] 01-architecture.md:145 - Block ID ^profile missing at end of paragraph
[WARNING] 03-decisions-log.md:67 - Wiki link [[fake-file]] target doesn't exist
[INFO] 02-design-questions.md:89 - Block ID ^q1 not descriptive
```

**Severity Levels:**
- **CRITICAL:** Broken functionality (broken links, malformed syntax)
- **WARNING:** Quality issues (missing display text, non-descriptive IDs)
- **INFO:** Style issues (formatting, consistency)

---

### Fix Recommendations

**For each issue, suggest:**
1. What's wrong (specific problem)
2. Why it matters (impact)
3. How to fix (specific action)
4. Example of correct form

---

## Integration with Workflow

**When to validate:**
- Immediate: After high-risk edits (block IDs, wiki links, structure changes)
- Session end: Comprehensive check of all edited files
- On demand: User requests "validate [file]"
- Periodic: Weekly project-wide validation

**How to use this guide:**
- Reference during validation
- Check specific syntax rules
- Verify test procedures
- Compare against examples
- Report issues using format

---

## Future Automation

**Potential automated tests:**
- Block ID format validation (regex)
- Wiki link target checking (file existence)
- Dataview syntax parsing
- Cross-reference mapping
- Duplicate detection
- Consistency analysis

**Current state:** Manual validation using this guide
**Future state:** Automated scripts + manual verification

---

## Quick Reference

**Validation priority:**
1. Block IDs (format, placement, uniqueness)
2. Wiki links (targets exist, display text)
3. Document structure (required elements)
4. Cross-references (integrity)
5. Dataview queries (syntax)
6. Content coherence (consistency)

**Most common issues:**
1. Block ID on same line as header
2. Missing display text in block refs
3. Generic block ID names
4. Broken wiki link targets
5. Overused callouts

**Quick fixes:**
1. Move block IDs to end of paragraphs
2. Add display text: `[[file#^block|text]]`
3. Rename to descriptive: `^component-topic`
4. Fix target or remove link
5. Reduce callout usage
