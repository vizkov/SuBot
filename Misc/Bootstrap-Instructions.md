# SuBot Project - Bootstrap Instructions

**Version:** Bootstrap 1.1  
**Status:** Reference Guide
**Last Updated:** 2026-01-01 20:44 IST  
**Purpose:** Minimal core rules + triggers to load detailed instructions

---

## Platform

**User Environment:** Windows (user's files stored on Windows filesystem)
**Claude Environment:** Linux container (bash_tool runs Linux commands)
**Base Path:** `H:\My Drive\subot-project\`
**Critical:** Use Filesystem tools to access user's files. bash_tool runs bash/Linux commands (NOT PowerShell).

---

## Core Rules

**1. Design Before Implementation**
- All architecture decisions must be resolved before building begins
- Use design documents to explore, decide, then build

**2. Surgical Edits Only**
- Use `str_replace` for targeted changes
- NEVER rewrite entire sections unless explicitly requested
- Flag issues before fixing - ask which to address

**3. Maintain Obsidian Syntax**
- Preserve wiki links: `[[file#^block-id|display text]]`
- Keep block IDs: `^descriptive-id` at END of paragraphs
- Maintain callouts: `> [!warning|info|important|question]`
- Preserve tags: `#architecture #component #decision`

**4. Tool Selection for File Operations**

**ALWAYS use Filesystem tools for:**
- Reading files: `Filesystem:read_text_file`
- Writing/editing files: `Filesystem:edit_file`, `Filesystem:write_file`
- Searching for files: `Filesystem:search_files`
- Directory operations: `Filesystem:list_directory`, `Filesystem:create_directory`
- File metadata: `Filesystem:get_file_info`

**How Filesystem tools work:**
- Access user's Windows files directly (H:\My Drive\subot-project\)
- Handle path translation automatically (Windows → internal format)
- Work seamlessly regardless of user's OS

**When searching within files:**
- Read file content with Filesystem:read_text_file
- Search in memory (JavaScript string methods)
- Calculate line positions for view_range
- Do NOT use bash grep or text processing commands

**bash_tool (rarely needed):**
- Runs in Linux container (NOT on user's Windows machine)
- Only use for: Complex logic, calculations, data processing
- Do NOT use for: File operations, path manipulation, text searching
- If needed, use Linux/bash commands (NOT PowerShell commands)

**5. Update Timestamps**
- On substantive changes: Use `time:get_current_time` with timezone "Asia/Calcutta"
- Format: "Last Updated: YYYY-MM-DD HH:MM IST"

---

## When to Read Extended Instructions

**Read Project-Instructions.md when:**
- User asks about protocols, workflows, or "how to do X"
- Creating/modifying document structure (need detailed rules)
- Uncertain about any rule or protocol
- Feedback processing AND (unsure of protocol OR document timestamp changed)
- Starting new workflow not used recently

**Read Obsidian-Guide.md when:**
- Creating new documents (need syntax examples)
- User asks about Obsidian features
- Uncertain about wiki link, block ID, callout, or tag syntax

**Don't read when:**
- Confident about current workflow
- Recently applied same protocol (within session)
- Document unchanged since last read (check timestamp)

**File paths:**
- `H:\My Drive\subot-project\Misc\Project-Instructions.md`
- `H:\My Drive\subot-project\Misc\Obsidian-Guide.md`

---

## Feedback Processing

**Trigger detection:**
```
"process design feedback" → Read H:\My Drive\subot-project\Misc\Feedback-Design.md
"process instructions feedback" → Read H:\My Drive\subot-project\Misc\Feedback-Instructions.md
"process feedback" → Ask user which context
```

**After reading feedback file:**
1. **Assess knowledge:** Do I know the current feedback processing protocol?
   - If uncertain about workflow, validation, or recent changes → Read Project-Instructions.md
   - If confident about protocol → Proceed with processing
2. Follow "Feedback Processing Protocol" section
3. Validate feedback structure (Type, Action, Document, Section)
4. Acknowledge briefly - do NOT repeat feedback content back
5. Ask which items to address
6. Make surgical edits
7. Confirm updates

**Note:** Project-Instructions.md timestamp helps assess if re-reading needed (check "Last Updated" vs last session)

---

## Block-Level Reading (Token Optimization)

**When user references specific sections:**

**If wiki link provided:**
```
User: [[01-architecture#^profile-interaction-model]]
→ Extract filename + block ID
→ Read file, search for "^profile-interaction-model"
→ Re-read with view_range: ±10 lines around block
```

**If natural language:**
```
User: "the profile interaction section"
→ Infer document from context
→ Read section only (not full file)
```

**Implementation:**
1. `Filesystem:read_text_file(file)` - get content
2. Search in memory for block ID or section name
3. Calculate line number
4. `Filesystem:read_text_file(file, view_range=[start, end])` - reread targeted lines

**Token savings:** ~90-95% (read ~21 lines vs 500+ line file for large documents)

---

## Document Structure

**Core documents:**
- 00-overview.md, 01-architecture.md, 02-design-questions.md
- 03-decisions-log.md, 04-roadmap.md, 05-future-considerations.md

**Component designs:** 2.1-component-designs/, 2.2-security/

**Meta documents:** Misc/ folder
- Project-Instructions.md (detailed workflows)
- Obsidian-Guide.md (syntax reference)
- Feedback-Design.md (design feedback - user maintains)
- Feedback-Instructions.md (workflow feedback - user maintains)
- Template.md (Templater-enabled feedback template)

---

## Template.md (Templater-Enabled)

**How it works:**
- Template.md uses Templater to auto-discover files and sections dynamically
- When user creates feedback in Obsidian, Templater runs and finds:
  - All .md files (excluding Template.md, Feedback-*.md, Archive/*)
  - All ## section headers in each file
- User selects from dropdowns, template populates feedback structure

**No manual updates needed for structure changes:**
- Template.md discovers structure automatically every time it runs
- Claude does NOT update Template.md when files/sections are added/renamed/deleted
- Adding/renaming files or sections → Template.md sees changes automatically

**Claude CAN update Template.md only when:**
- Templater code has bugs or errors
- User explicitly requests changes to template functionality
- Exclusion list needs updating (adding new file patterns to exclude)
- Template structure/format needs modification

**Files excluded (hardcoded in template):**
- Template.md, Feedback-*.md, Archive/*

---

## Obsidian Syntax Requirements

**When creating/editing any document:**

**Always add:**
- Document tags at top: `#architecture`, `#decision`, `#component`, etc.
- Block IDs after: ADRs (`^adr-topic`), major sections (`^section-name`), key content
- Wiki links when: Referencing other documents/sections/decisions
- Callouts for: Warnings, critical notes, open questions
- Related links at end of sections

**Block ID format:**
- Use lowercase with hyphens: `^component-topic-description`
- Place at END of paragraph (invisible in Obsidian reading mode)
- For headers: Separate line after header

**Wiki link format:**
- Always use display text: `[[file#^block|readable text]]`
- NOT: `[[file#^block]]` (shows technical ID)

---

## Emergency Protocols

**File not found:**
1. Try alternative path
2. Ask user to verify file location
3. Request manual upload if needed

**Command fails:**
1. Use Filesystem tool alternative (primary method for file operations)
2. If bash_tool needed, use Linux/bash commands
3. Report specific error to user

**Block ID not found:**
1. Fall back to reading full section
2. Notify user block ID missing
3. Continue with available context

**Syntax error (Dataview, etc.):**
1. Flag error with line reference
2. Do NOT break document
3. Ask user to fix or provide correction

---

## Quality Checks

**Before completing any session:**
- All acknowledged feedback items addressed or explicitly deferred?
- All edits used str_replace (no full rewrites)?
- All new/edited content has Obsidian syntax?
- Timestamps updated where appropriate?

---

## Key Principle

**This bootstrap = Stable core rules only**

All detailed protocols, examples, edge cases, and workflows live in Project-Instructions.md and Obsidian-Guide.md.

When uncertain about anything: Read the appropriate extended instructions file.

User rarely updates this bootstrap. Most changes happen in the extended files.

---

## When to Update This Bootstrap

**User updates bootstrap ONLY when:**
- Platform changes (Windows → Linux)
- File structure reorganizes (base paths change)
- Core rule changes (fundamental behavior modification)
- New critical automation added (must be in bootstrap)
- Emergency protocol changes (failure handling updates)

**User does NOT update bootstrap when:**
- Detailed protocols change (lives in Project-Instructions.md)
- Examples updated (lives in Obsidian-Guide.md)
- Edge cases clarified (lives in extended files)
- New workflows added (lives in Project-Instructions.md)
- Token optimization strategies refined (lives in extended files)
- Feedback processing details change (lives in Project-Instructions.md)

**How to know if bootstrap needs updating:**
- Claude will explicitly say: "This change requires bootstrap update"
- User will be notified with specific section to update

**Version tracking:**
- Current bootstrap version: 1.1
- Last updated: 2026-01-01 20:44 IST
- When bootstrap changes, version increments and "Last Updated" changes
