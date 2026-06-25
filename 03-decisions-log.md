# SuBot - Decisions Log

**Last Updated:** 2025-12-15 18:37 IST
**Status:** Review

#decision

---

## Decisions Log ^decisions-log

### 2025-12-14 02:30 IST - CLI as Primary Interface
^adr-cli-primary-interface
**Question**: What interface should we build first?

**Options Considered**: 
- CLI tool
- Web-based UI
- Desktop GUI (Electron)

**Decision**: CLI tool is the primary/initial interface

**Rationale**: 
- Faster to develop and iterate
- Better for automation and scripting
- Aligns with AppSec engineer workflows
- Lower complexity for MVP

**Implications**: 
- Architecture must support future GUI layer
- Core logic should be interface-agnostic
- Electron UI/UX planned for late-stage (possibly during/after experimental features)

**Related:** [[01-architecture#^cli-interface|CLI Interface design]]

---

### 2025-12-14 02:45 IST - Profile Interaction Model
^adr-profile-interaction
**Question**: How should the 9 AI profiles interact with each other?

**Options Considered**:
- Option A: Message Bus only (pub/sub events)
- Option B: Shared Context only (Security-Context Map)
- Option C: Orchestrator-Mediated only (centralized hub)

**Decision**: Implement all three mechanisms

**Rationale**: Each serves a distinct purpose:
- Message Bus maintains workflow order and event-driven behavior
- Shared Context enables knowledge accumulation and learning
- Orchestrator manages complex workflows and conflict resolution

**Implications**:
- More complex architecture
- Need clear rules for when each mechanism is used
- Orchestrator profile becomes central coordination point

**Related:** [[01-architecture#^profile-interaction-model|Profile interaction model]]

---

### 2025-12-14 02:45 IST - Profile Activation Model
^adr-profile-activation
**Question**: When should each AI profile activate?

**Options Considered**:
- Option A: Always On (all 9 profiles analyze everything)
- Option B: Triggered (events trigger specific profiles)
- Option C: User-Selectable (user controls active profiles)

**Decision**: Option B - Event-triggered activation

**Rationale**:
- Most efficient resource usage
- Allows dynamic response to assessment needs
- Avoids expensive always-on operation
- Preserves profiles as "rails" (not user-controllable)

**Implications**:
- Need clear event → profile mapping
- Orchestrator manages activation logic
- Profile state must persist between activations

**Related:** [[01-architecture#^profile-activation-model|Profile activation details]]

---

### 2025-12-14 02:45 IST - Conflict Resolution Hierarchy
^adr-conflict-resolution
**Question**: How should the system handle disagreements between profiles?

**Options Considered**:
- Pure voting system
- Orchestrator decides everything
- User approves everything
- Enforcer has veto power

**Decision**: Weighted hierarchy: User >>> Enforcer > Orchestrator

**Rationale**:
- User must have final authority (human-in-the-loop principle)
- Enforcer represents methodology/compliance guardrails
- Orchestrator mediates between other profiles
- Clear escalation path prevents deadlocks

**Implications**:
- Need conflict detection mechanism
- Escalation UI for user decisions
- Enforcer rules must be well-defined

**Related:** [[01-architecture#^profile-conflict-resolution|Conflict resolution details]]

---

### 2025-12-14 02:45 IST - Profile Memory/State
^adr-profile-memory
**Question**: Should profiles maintain their own memory and state?

**Options Considered**:
- Stateless profiles (fresh each invocation)
- Shared state only (Security-Context Map)
- Individual profile state + shared state

**Decision**: Profiles maintain individual state + shared state

**Rationale**:
- Essential for learning and improvement
- Prevents repetitive work (e.g., re-validating same finding)
- Enables personalization (e.g., learning user preferences)
- Supports skill progression tracking

**Implications**:
- Profile state stored in Security-Context Map
- State persistence depends on operating mode
- Need state serialization/deserialization

**Related:** [[01-architecture#^profile-state-memory|Profile state details]]

---

### 2025-12-14 02:45 IST - Methodology Configuration Format
^adr-methodology-config
**Question**: How should methodologies be specified?

**Options Considered**:
- JSON
- YAML
- Python code
- Custom DSL

**Decision**: YAML with flexible base structure

**Rationale**:
- Human-readable and editable
- Supports comments for documentation
- Flexible enough for complex configurations
- Industry-standard for configuration

**Implications**:
- Need YAML parser and validator
- Base schema must be extensible
- Support for custom fields

**Related:** [[01-architecture#^methodology-config-format|Methodology config details]]

---

### 2025-12-14 02:45 IST - Data Representation Conversion
^adr-data-conversion
**Question**: Should different data representations (object/graph/DAG/compressed) be convertible?

**Options Considered**:
- Fixed representation per data type
- Manual conversion when needed
- Automatic bidirectional conversion

**Decision**: Yes, representations must be convertible

**Rationale**:
- Enables multi-view analysis
- Different profiles need different representations
- Graph analysis benefits from seeing findings as nodes
- LLM efficiency requires compression

**Implications**:
- Need conversion functions for all representation pairs
- Potential data loss during compression (must be handled)
- Conversion triggers must be defined

**Related:** [[01-architecture#^internal-data-model|Internal data model]]

---

### 2025-12-14 02:45 IST - Skills System Structure
^adr-skills-structure
**Question**: How should the skills system be organized?

**Options Considered**:
- Flat skill list
- Category-based grouping
- Hierarchical skill tree with prerequisites

**Decision**: Hierarchical skill tree with fundamentals and role-specific branches

**Rationale**:
- Matches learning progression (fundamentals → specialized)
- Supports different assessment types (Pentest, SecArch, etc.)
- Enables skill prerequisite tracking
- Inspired by GitHub Skills folder structure

**Implications**:
- Skills organized: Fundamentals → Role-Specific → Domain-Specific
- Prerequisite validation before skill acquisition
- Different methodologies emphasize different skill sets
- Skill mastery tracked numerically (0-100)

**Related:** [[01-architecture#^dag-continuous-learning|DAG Continuous Learning]]
