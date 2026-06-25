# Obsidian Syntax Guide

**Last Updated:** 2026-01-01 20:47 IST
**Status:** Reference Guide

#workflow #reference

---

## Overview

This guide provides comprehensive details on using Obsidian syntax in the SuBot project. For quick reference, see [[Project-Instructions#Obsidian Syntax Usage]].

**Enabled Features:**
- Wiki links & backlinks
- Block references
- Callouts
- Embeds for diagrams
- Tags for categorization
- Templater compatibility

---

## Wiki Links

**Purpose:** Cross-reference documents and sections

**Syntax:**
```markdown
[[filename]]                          # Link to document
[[filename#Section Name]]             # Link to section
[[filename#Section Name|display]]     # Link with custom text
[[filename#^block-id]]                # Link to specific block
[[filename#^block-id|display]]        # Block link with custom text
```

**When to use:**
- Referencing other design documents
- Citing decisions from decisions log
- Pointing to related questions
- Cross-referencing components

**Examples:**

**Good:**
```markdown
See [[01-architecture#^chassis-plugin-system|chassis plugin system]] for implementation details.

This decision relates to [[03-decisions-log#^adr-profile-interaction|profile interaction decision]].

For security implications, refer to [[2.2-security/security-considerations#Threat Model]].
```

**Avoid:**
```markdown
See the architecture document for chassis details. (Too vague, no link)
See 01-architecture.md section on chassis. (Plain text, not clickable)
```

---

## Block References

**Purpose:** Create precise citation points for important content

**Naming Convention:** Descriptive block IDs
- Format: `^component-topic-description`
- Examples: `^adr-microkernel-decision`, `^chassis-msg-bus-design`
- Use lowercase with hyphens
- Be specific and meaningful

**Placement Rules:**
- For paragraphs: At END of paragraph text (invisible in reading mode)
- For headers: On separate line after header
- For lists: After the list ends

**When to add block IDs:**
- Every ADR: `^adr-{topic}`
- Key questions: `^question-{topic}`
- Major sections: `^{component}-{aspect}`
- Security considerations: `^security-{topic}`

**Examples:**

**Correct placement (invisible in reading mode):**
```markdown
## ADR-001: Micro-Kernel Architecture
^adr-microkernel-architecture

Decision: Use plugin-based micro-kernel with minimal chassis.

Rationale: Enables modularity and tool-agnostic design. ^rationale-microkernel
```

**Linking to blocks:**
```markdown
As decided in [[03-decisions-log#^adr-microkernel-architecture|microkernel decision]], we use plugins.

See [[01-architecture#^chassis-plugin-system|plugin system design]] for details.
```

---

## Callouts

**Enabled Types:**
- `[!warning]` - Security risks, breaking changes
- `[!info]` - Additional context, background
- `[!important]` - Critical points, key takeaways
- `[!question]` - Open questions, needs discussion

**Usage guidelines:**
- Use sparingly (too many callouts reduce impact)
- Keep content concise (2-4 lines typical)
- Place near relevant content
- Title should be specific and actionable

**Examples:**

```markdown
> [!warning] Security Implication
> Storing credentials in Security-Context Map requires encryption at rest.

> [!info] Design Context
> This decision was influenced by the tool-agnostic principle.

> [!important] Implementation Requirement
> All profiles must implement the `process()` method.

> [!question] Needs Decision
> Should plugins support hot-reload during development?
```

---

## Tags

**Tag Taxonomy:**

**Core categories:**
- `#architecture` - Architecture decisions, system design
- `#security` - Security considerations, threat models
- `#decision` - ADRs, key choices made
- `#question` - Unresolved questions
- `#workflow` - Process, protocols

**Component-specific:**
- `#component` - General component tag
- `#component/chassis` - Chassis-specific
- `#component/profile` - AI profile designs
- `#component/adapter` - Tool adapters

**Document type:**
- `#overview` - High-level vision
- `#roadmap` - Implementation planning
- `#future` - Deferred items

**Where to place:**
- Document top (after title/metadata)
- Section level (after section header)
- Inline within content

**Examples:**

```markdown
# Component Design

**Last Updated:** 2025-12-15 15:00 IST

#architecture #component/chassis #decision

---

## Plugin System Design

#component #security

The plugin system provides...
```

---

## Templater Compatibility

**What Claude should preserve:**
- Templater syntax: `<% %>` or `<%* %>`  
- Dynamic expressions: `<% tp.date.now() %>`
- Suggester calls: `<% tp.system.suggester(...) %>`
- Cursor placement: `<% tp.file.cursor(1) %>`

**When editing Template.md:**
- Preserve all `<% %>` blocks exactly
- Do not execute or interpret Templater code
- Only edit surrounding content

---

## Block-Level Context Loading

**Purpose:** Read only referenced blocks instead of entire files (90-95% token savings).

**Process:**
1. User provides: `[[file#^block-id]]`
2. Extract: filename + block ID
3. Read file: `Filesystem:read_text_file(file)` to load content
4. Search in memory: Find `^block-id` pattern, calculate line number
5. Re-read with view_range: `Filesystem:read_text_file(file, view_range=[start, end])` for ±10 lines
6. Savings: ~200 tokens vs ~5000 for full file

**Example:**
```
User: "See [[01-architecture#^chassis-plugin-system]]"

Claude:
1. Reads 01-architecture.md into memory
2. Searches for ^chassis-plugin-system → finds at line 145
3. Re-reads with view_range=[135, 155] (21 lines)
4. Processes with ~200 tokens vs ~5000 for full file
5. Token savings: 96%
```

**Implementation notes:**
- Always read ±10 lines for context
- If block near file start/end, adjust range accordingly
- If multiple matches, use first + flag for user
- If not found, fall back to full file + notify user
- Use Filesystem tools for reading (works on Windows user files)
- Do NOT use grep/bash commands for file searching

---

## Common Mistakes

**1. Block reference visible in reading mode:**
```markdown
❌ Wrong:
**Term** ^block-id
Definition here.

✓ Correct:
**Term**
Definition here. ^block-id
```

**2. Missing display text in block links:**
```markdown
❌ Shows: "See 03-decisions-log > ^adr-001 for details"
[[03-decisions-log#^adr-001]]

✓ Shows: "See microkernel decision for details"  
[[03-decisions-log#^adr-001|microkernel decision]]
```

**3. Too many callouts:**
```markdown
❌ Callout overload (loses impact)

✓ Selective use - only for critical info
```

**4. Non-descriptive block IDs:**
```markdown
❌ Generic: ^block1, ^section-a
✓ Descriptive: ^adr-microkernel-decision, ^chassis-plugin-loading
```

---

## Dataview Patterns

**Common patterns for dynamic lists:**

**1. File listing with metadata:**
```markdown
\```dataview
TABLE file.mtime as "Last Modified", file.size as "Size"
FROM ""
WHERE file.name != "Template"
SORT file.name ASC
\```
```

**2. Status dashboard:**
```markdown
\```dataview
TABLE status as "Status", file.mtime as "Last Updated"
FROM ""
WHERE status
GROUP BY status
\```
```

**3. Tag-based listing:**
```
\dataview
LIST
FROM #architecture
SORT file.name
```

**4. Recent activity:**
```markdown
\```dataview
TABLE file.mtime as "Updated"
FROM ""
SORT file.mtime DESC
LIMIT 10
```

**Usage notes:**
- Claude never edits Dataview blocks
- Claude can add NEW queries when appropriate
- If query not working, Claude flags for user to fix
