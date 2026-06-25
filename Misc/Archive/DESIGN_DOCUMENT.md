# SuBot - AI AppSec Engineer Design Document

**Project Goal**: Build an AI tool that mimics an AppSec engineer with modular architecture, practical execution, and continuous learning capabilities.

**Document Purpose**: This is our master reference for all design discussions, decisions, and rationale. It will be updated continuously as we work through each component.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Glossary](#glossary)
3. [Core Design Principles](#core-design-principles)
4. [System Architecture](#system-architecture)
5. [Component Designs](#component-designs)
6. [Design Questions & Decisions](#design-questions--decisions)
7. [Decisions Log](#decisions-log)
8. [Implementation Roadmap](#implementation-roadmap)
9. [Future Considerations](#future-considerations)
10. [Revision History](#revision-history)

---

## Project Overview

### Vision
An AI-powered AppSec engineer that:
- Operates as a CLI tool (with future Electron UI/UX expansion planned)
- Focuses on DAST initially
- Requires human approval for all operations
- Stores everything locally
- Learns and improves continuously
- Adapts to any security tool or methodology

### Deployment Model
- **Interface**: Command-line tool (current), Electron-based GUI (future)
- **Primary Focus**: DAST (Dynamic Application Security Testing)
- **Approval Model**: Human-in-the-loop for all operations
- **Storage**: Local filesystem/database
- **LLM Strategy**: Hybrid (OSS for expensive ops, paid for reasoning)

---

## Glossary

### Core Concepts

**Chassis**: The minimal core of the system that handles input → AI → output. All other functionality plugs into it.

**Profiles**: Independent AI personas that act as critics and contributors, not sequential workflow stages. They operate concurrently on data/operations.

**Methodology**: Configuration that defines the approach, goals, and guidelines for a specific type of security assessment (e.g., Pentest vs Threat Modeling).

**Assessment Lifecycle**: Framework of phases (Purpose, Requirement, Scope, etc.) that guide but don't rigidly constrain the assessment process.

**Security Context/Map**: Persistent or ephemeral knowledge base that enriches over time with findings, patterns, and learned information.

**DAG Continuous Learning**: Directed Acyclic Graph representing skill dependencies and progression, inspired by character progression systems in games.

**Skills**: Concrete, actionable capabilities (e.g., "SQL Injection Tester", "Firewall Rule Automator") organized hierarchically and progressively.

**Tool Adapter**: Generic interface that normalizes any security tool's output into the system's internal format.

**Normalizer**: Component that converts diverse tool outputs into consistent internal data representations.

**Polymorphic Data Model**: Context-dependent data representation (object/graph/compressed/DAG) that adapts based on operational needs.

### AI Profiles

**Evaluator**: Assesses the validity and severity of actions/findings from other profiles.

**Validator**: Independently verifies correctness of outputs from other profiles.

**Enforcer**: Ensures compliance with methodology rules, standards, and constraints.

**Recommender**: Suggests next actions, techniques, or tools to other profiles.

**Orchestrator**: Manages workflows, action queues, and message passing between profiles.

**Builder**: Constructs payloads, test cases, and exploit code based on validated findings.

**Debugger**: Diagnoses tool failures, scan errors, and operational issues.

**Synthesizer**: Aggregates findings into coherent security narratives and reports.

**Optimizer**: Identifies and eliminates redundancy, reduces token usage, streamlines operations.

### Operating Modes

**Static Mode (Pro-context)**: Full persistence of learning, findings, and context across sessions.

**Ephemeral Mode (Blind)**: No persistence; each session starts fresh with no historical context.

**Mixed Mode**: Configurable selective persistence (e.g., keep methodology learnings but not target-specific data).

---

## Core Design Principles

### 1. Modularity = Tool Agnostic
- No fixed tool integrations
- System accepts ANY tool output (Nuclei, Burp, ZAP, custom scripts)
- Input adapters normalize diverse formats to internal representation
- "Chassis" doesn't care about specific tools

**Design Implication**: Generic input parsers, normalized internal data model

**Example**: Whether you feed it Nuclei JSON, Burp XML, or ZAP output, the system normalizes it to a standard "Finding" object with fields like severity, location, evidence, etc.

### 2. Micro-kernel Architecture
- Minimal core ("chassis" only)
- Everything else is pluggable
- Methodologies are configurations, not hardcoded logic
- Assessment lifecycle is a framework, not rigid steps

**Design Implication**: Clear plugin interfaces, configuration-driven behavior

**Example**: To add support for a new methodology like "PCI-DSS Compliance Testing", you create a YAML config file—no code changes to the core system.

### 3. AI as Chassis
- AI is the operating system for AppSec operations
- Tools produce raw data
- AI interprets, contextualizes, and decides actions
- 9 profiles act as specialized system processes

**Design Implication**: AI orchestration layer is the core runtime

**Example**: A DAST scan returns 50 potential SQLi findings. The Evaluator assesses which are true positives, the Validator attempts to reproduce them, the Recommender suggests deeper testing on confirmed findings, and the Orchestrator coordinates these activities.

### 4. Deliberate Before Action
- Discussion and design precede implementation
- Questions must be asked AND answered
- Unbiased evaluation of options
- Use existing solutions, don't reinvent

**Design Implication**: This document is critical; implementation happens only after design agreement

---

## System Architecture

### High-Level Data Flow
```
Input (any tool/format) 
  ↓
[Normalizer] - converts to internal format
  ↓
[Security Context/Map] - enriches with history/knowledge
  ↓
[AI Chassis with active profile(s)] - reasons about it
  ↓
[Action/Recommendation] - with justification/logs
  ↓
[User Approval Gate]
  ↓
[Execution/Storage] - updates context, triggers next cycle
```

### Core Components

#### 1. CLI Interface & UX
*Status: Design in progress*

**Current Focus**: Command-line interface
**Future Expansion**: Electron-based UI/UX (planned for late-stage development, possibly during/after experimental features)

**Pending Design Decisions:**
- Command vs conversational vs hybrid model
- Session management approach
- Approval UX pattern

#### 2. Micro-kernel Architecture
*Status: Concept defined*

**Design**: 
- Minimal chassis handles: input → AI → output
- All other functionality via plugins/modules
- Configuration-driven behavior

**Example Structure**:
```python
class Chassis:
    def process(self, input_data):
        normalized = self.normalizer.normalize(input_data)
        enriched = self.context_map.enrich(normalized)
        result = self.ai_profiles.process(enriched)
        if self.approval_gate.request_approval(result):
            return self.execute(result)
```

#### 3. AI Profile System (9 Personas)
*Status: Design in progress*

**Profiles are independent personas, not workflow stages.**

**Key Design Decision**: Profiles operate concurrently as critics and contributors, not sequentially.

| Profile | Primary Function | Example Behavior |
|---------|------------------|------------------|
| **Evaluator** | Evaluates actions taken by other profiles | "This SQL injection claim needs evidence. Confidence: 60%" |
| **Validator** | Validates correctness of other profiles' outputs | "I reproduced the SQLi with payload X. Confirmed." |
| **Enforcer** | Ensures compliance with methodology and standards | "Pentest methodology requires 3 confirmations for critical findings" |
| **Recommender** | Provides suggestions to other profiles | "Consider testing adjacent /api/orders endpoint for same vulnerability" |
| **Orchestrator** | Manages workflows/actions/queues/messages between profiles | "Queuing validation task, prioritizing by severity" |
| **Builder** | Constructs payloads/test cases based on validated findings | "Generated 5 SQLi payloads targeting MySQL backend" |
| **Debugger** | Diagnoses and fixes failures in scans/tools | "Nuclei scan failed: rate limit hit. Retrying with 2s delay" |
| **Synthesizer** | Creates coherent security narrative across findings | "15 SQL injection points suggest systematic input validation failure" |
| **Optimizer** | Streamlines operations, reduces redundancy/waste | "Detected 8 duplicate scans on same endpoint. Consolidating..." |

**Interaction Model** (from user input):
- **Decision**: All three interaction options serve a purpose and will be implemented
  - **Message Bus**: Maintains workflow order and event-driven activation
  - **Shared Context**: Profiles read/write to Security Context/Map directly
  - **Orchestrator-Mediated**: Orchestrator manages queuing and prioritization

**Activation Model** (from user input):
- **Decision**: Option B - Triggered activation is preferred
  - Certain events trigger specific profiles
  - More efficient than always-on
  - Examples:
    - New finding → Evaluator + Validator
    - Scan failure → Debugger
    - User asks "what next?" → Recommender

**Conflict Resolution** (from user input):
- **Decision**: Weighted voting with user override
  - Hierarchy: **User >>> Enforcer > Orchestrator**
  - Certain things get vetoed automatically by Enforcer
  - User has final say on all conflicts

**State/Memory** (from user input):
- **Decision**: Yes, profiles maintain their own memory/state
  - Validator remembers what it's already validated
  - Recommender tracks accepted/rejected recommendations
  - Optimizer learns which optimizations worked
  - This is vital for avoiding repetition and improving over time

#### 4. Methodology Framework
*Status: Design decisions made*

**Methodologies supported:**
- Pentest
- Payment (PCI/payment security)
- SCR (Secure Code Review)
- SDR (Security Design Review)
- TM (Threat Modeling)
- Architecture Review
- Malware Analysis
- Red Team
- Blue Team
- Combo (hybrid approaches)

**Methodology Purpose** (from user input):
- Define approach and goals for security review
- Inform which techniques/tools are appropriate
- **Do NOT** affect profile behavior (profiles are "rails" for the system)
- Every profile consumes methodology as input and acts on it differently

**Configuration Format** (from user input):
- **Decision**: YAML with flexible base structure
- Base structure can be expanded/customized

**Example Methodology (Pentest)**:
```yaml
name: "Penetration Test"
approach: "offensive"
goals:
  - "Identify exploitable vulnerabilities"
  - "Demonstrate business impact"
  - "Provide remediation guidance"
guidelines:
  - "Follow OWASP Testing Guide"
  - "Require proof-of-concept for high/critical"
  - "No testing in production without explicit approval"
phases:
  - recon
  - vulnerability_discovery
  - exploitation
  - post_exploitation
  - reporting
tool_preferences:
  dast: ["nuclei", "zap"]
  network: ["nmap", "masscan"]
```

**Enforcer Profile Usage** (from user input):
- **Decision**: Enforcer ensures compliance with methodology
  - Checks action compliance with guidelines
  - Verifies goals are being addressed
  - Ensures phases are followed (flexibly, not strictly)
  - **But**: All profiles consume methodology, not just Enforcer

**Custom Methodologies** (from user input):
- **Decision**: Yes, users can define custom methodologies

**Methodology Combination** (from user input):
- **Decision**: Yes, methodologies can be hybridized (e.g., "Red Team + Threat Model")

**Methodology Evolution** (from user input):
- **Decision**: Can be suggested to user based on findings, then approved
- Example: "Based on architecture findings, recommend adding Threat Modeling phase"

**Methodology Effects** (from user input):

| Aspect | How Methodology Affects It |
|--------|---------------------------|
| **Tool recommendations** | Based on OSS availability, user-defined preferences, or web search (with approval) |
| **Finding prioritization** | User input + profile guidance initially |
| **Security Context/Map** | Only affects operating mode (Static/Ephemeral/Mixed). Doesn't dictate, but can be enriched by methodology output if user approves |
| **Profile activation** | Falls more on Assessment Lifecycle than Methodology. All methodologies use approximately same lifecycle phases |

#### 5. Assessment Lifecycle
*Status: Framework defined*

**Phases:**
- Purpose
- Requirement
- Scope
- Restrictions
- Pre-requisites
- Pre-Qualification
- Scan
- Triage
- QA/QC
- Observations

**Note**: This is a framework, not rigid sequential steps

**Example Flow**:
```
User starts assessment → Purpose (define goals) → Scope (target boundaries)
→ Restrictions (what's off-limits) → Pre-requisites (access, credentials)
→ Scan (execute tools) → Triage (categorize findings)
→ QA/QC (validate, prioritize) → Observations (document, report)
```

Assessment can loop back (e.g., Triage reveals new Scope requirements).

#### 6. Security Context/Map
*Status: Concept defined*

**Operating Modes:**
- Static (Pro-context mode) - persistent learning
- Ephemeral (Blind-mode) - no persistence
- Mixed-configurable (Pick and choose)

**Functions:**
- Enriches profile knowledge
- Enriches methodology application
- Enriches DAG continuous learning
- Can generate/use metadata tags

**Storage**: Local (files/database)

**Example Data Stored**:
```json
{
  "target": "api.example.com",
  "tech_stack": ["Python", "Flask", "PostgreSQL"],
  "historical_findings": [
    {"type": "SQLi", "endpoint": "/api/login", "severity": "high", "date": "2024-12-01"}
  ],
  "skill_mastery": {
    "sql_injection_detection": 85,
    "xss_detection": 72
  },
  "patterns": {
    "common_vuln_types": ["SQLi", "IDOR", "CSRF"]
  }
}
```

#### 7. Tool Adapter System
*Status: Concept defined*

**Design**: Generic adapters for any tool output, normalize to internal data format(s)

**Starting focus**: DAST tools (Nuclei, ZAP, Burp)

**Example Adapter Interface**:
```python
class ToolAdapter:
    def parse(self, raw_output: str) -> List[Finding]:
        """Convert tool-specific output to Finding objects"""
        pass
    
    def validate(self, finding: Finding) -> bool:
        """Verify finding data is complete"""
        pass
```

#### 8. Internal Data Model
*Status: Design decisions made*

**Key Insight**: Data model is polymorphic, context-dependent

**Representation Decision Logic** (from user input):
- **Decision**: Depends on whether we use AI/Agentic vs conventional approach
- If AI-driven: Relies on AI/Agentic/RAG patterns to choose representation
- If conventional: Explicit decisions for each use-case
- Component-specific design will clarify this further

**Representation Types**:
- Finding from tool → **Object** representation
- Attack surface mapping → **Graph** representation
- Token optimization → **Compressed** representation
- Skill progression → **DAG** representation

**Conversion Between Representations** (from user input):
- **Decision**: Yes, representations need to convert between each other
- Finding object CAN become a node in the attack graph
- Graph patterns CAN collapse into compressed tokens for LLM
- This enables information exchange and multi-view analysis

**Example Object → Graph Conversion**:
```python
# Finding object
finding = Finding(
    type="SQLi",
    endpoint="/api/login",
    parameter="username",
    severity="high"
)

# Convert to graph node
graph.add_node(
    id="finding_001",
    type="vulnerability",
    data=finding,
    edges=[
        ("finding_001", "endpoint_/api/login"),
        ("finding_001", "attack_surface")
    ]
)
```

#### 9. DAG Continuous Learning
*Status: Framework defined, Skills system clarified*

**Inspired by**: Character Progression System design patterns

**Skills System** (informed by GitHub repo):

Skills are **concrete, actionable capabilities** organized hierarchically:

```
Fundamentals/
  ├── Languages (Python, Go, etc.)
  ├── OS and Networking
  ├── Programming
  └── Design and Architecture

Role-Specific Skills/
  ├── AI Pentester (100 skills)
  │   ├── Reconnaissance (8 skills)
  │   ├── Exploitation (7 skills)
  │   ├── Post-Exploitation (6 skills)
  │   └── ...
  ├── AI Engineer (65 skills)
  │   ├── Network Security (10 skills)
  │   ├── Web App Security (10 skills)
  │   └── ...
  ├── Security Architect (100 skills)
  └── Product Security (100 skills)
```

**Example Skills**:
- "Firewall Rule Automator"
- "SQL Injection Exploiter"
- "Threat Intelligence Feed Integration"
- "Certificate Pinning Enforcer"

**Skill Characteristics**:
- Have prerequisites (e.g., must know "Port Scanning" before "Banner Grabbing")
- Can be leveled up through practice
- Some have checkboxes for mastery tracking
- Different methodologies emphasize different skills

**Learning Capabilities**:
- Can attain and hone skills
- Pre-configured skill sets
- Identifies takeaways from experiences → skill progress
- Identifies skill gaps in existing skillset
- Looks for new skills to acquire

**Learning Sources**:
- **Active**: Labs, CTFs, Challenges (hands-on practice)
- **Passive**: CTI reports, Research Papers, Bug Bounty reports (knowledge acquisition)

**Configuration**: Highly configurable learning sources

**Example Skill Progression**:
```yaml
skill: "SQL Injection Detection"
category: "Web Application Security"
prerequisites: 
  - "HTTP Fundamentals"
  - "Database Basics"
current_level: 72/100
experience_sources:
  - ctf_challenge: "SQLi Lab 5" (2024-11-15) → +8 XP
  - bug_bounty_report: "Uber SQLi Analysis" (2024-12-01) → +5 XP
  - assessment: "Target ABC SQLi found" (2024-12-10) → +10 XP
next_level_requirements:
  - Complete 3 more CTF challenges OR
  - Successfully identify SQLi in 5 live assessments
```

#### 10. Hybrid LLM Strategy
*Status: Concept defined*

**Two-tier approach:**
- **OSS LLM**: Expensive routines (processing memory, context building, repetitive analysis)
- **Paid LLM** (Claude, GPT-4): Complex reasoning, validation, synthesis
- **Goal**: Reduce tokens sent to paid LLM

**Token Optimization** (Experimental):
- Noise reduction, redundancy elimination
- Metadata extraction, deduplication
- Rainbow table compacting of context/tokens

**Example Flow**:
```
1. OSS LLM processes 1000 findings → identifies patterns → compresses to summary
2. Paid LLM receives compressed summary + top 10 critical findings
3. Paid LLM performs deep reasoning on critical items
4. Result: 90% reduction in paid LLM tokens
```

#### 11. Approval System
*Status: Requirements defined*

**Requirement**: Human approval required for ALL operations

**Pending**: Pre-approval vs step-by-step vs trust levels

---

## Component Designs

### [To be filled in as we design each component]

---

## Design Questions & Decisions

### A. Internal Data Model

**Q1**: How does the system decide which data representation to use (object/graph/compressed/DAG)?

**Answer**: Depends on whether we use AI/Agentic vs conventional approach:
- **AI/Agentic approach**: System relies on AI/RAG patterns to dynamically choose representation based on context
- **Conventional approach**: Explicit decisions for each use-case
- Will be clarified during component-specific design

**Q2**: Do these representations need to convert between each other?

**Answer**: **Yes, conversions are needed**
- Finding object CAN become a node in the attack graph
- Graph patterns CAN collapse into compressed tokens for the LLM
- Without conversions, we lose ability to exchange information across representations

**Implications**: 
- Need conversion functions between all representation types
- Graph ↔ Object ↔ Compressed ↔ DAG
- Enables multi-view analysis and different consumption patterns

---

### B. AI Profile Interaction Model

**Q3**: How do profiles interact with each other?

**Answer**: **All 3 options serve a purpose - implement all**
- **Option A - Message Bus**: Maintains workflows and order (pub/sub events)
- **Option B - Shared Context**: Directly contributes to the system's feature/goal (all read/write to Security Context/Map)
- **Option C - Orchestrator-Mediated**: Definitely the Orchestrator's job (centralized hub)

**Rationale**: Each serves different aspects:
- Message Bus for event-driven triggers
- Shared Context for knowledge accumulation
- Orchestrator for workflow management

**Q4**: When does each profile activate?

**Answer**: **Option B - Triggered (events trigger specific profiles)**

**Rationale**: Optimal approach balancing efficiency and coverage
- New finding → Evaluator + Validator
- Scan failure → Debugger  
- User query "what next?" → Recommender
- Not always-on (too expensive), not user-selectable (defeats profiles as "rails")

**Q5**: What happens when profiles disagree?

**Answer**: **Weighted vote with veto hierarchy**

**Decision Hierarchy**: **User >>> Enforcer > Orchestrator**
- User has ultimate authority
- Enforcer can veto on methodology/compliance violations
- Orchestrator mediates between other profiles
- Certain rules get automatic veto (e.g., Enforcer blocks critical findings without validation)

**Example Scenario**:
```
Evaluator: "This is high severity"
Validator: "I can't reproduce, likely false positive"  
Enforcer: "Methodology requires 3 confirmations for high severity"
→ Orchestrator: "Requesting additional validation"
→ If still no consensus → Escalate to User
```

**Q6**: Do profiles maintain their own memory/state?

**Answer**: **Yes, vital for system effectiveness**

Specific behaviors:
- **Validator** remembers what it's already validated (avoid re-validation)
- **Recommender** tracks accepted/rejected recommendations (learn preferences)
- **Optimizer** learns which optimizations worked (improve over time)
- **Evaluator** tracks false positive patterns
- **Builder** remembers successful payload patterns

**Implications**:
- Each profile needs persistent storage for its state
- State should be part of Security Context/Map
- State persists based on operating mode (Static/Ephemeral/Mixed)

---

### C. Methodology Framework

**Q7**: How is a methodology specified/structured?

**Answer**: **YAML format with flexible base structure**

**Base elements that can be expanded/customized**:
```yaml
name: str
approach: str  # offensive, defensive, compliance, etc.
goals: List[str]
guidelines: List[str]
phases: List[str]
tool_preferences: Dict[str, List[str]]
compliance_requirements: Optional[Dict]
custom_fields: Optional[Dict]  # User extensibility
```

**Q8**: Should methodologies define which profiles are emphasized?

**Answer**: **No, profiles are not affected by methodology**

**Rationale**: Profiles operate on/with each other's output. They act like "rails" for the system.
- Profiles do not vary based on input
- All profiles consume methodology as input
- Each profile acts on methodology differently
- Methodology doesn't dictate which profiles are active

**Q9**: How does the Enforcer profile use methodology?

**Answer**: **Enforcer ensures compliance, but ALL profiles consume methodology**

Enforcer-specific actions:
- Check action compliance with guidelines ✓
- Verify goals are being addressed ✓
- Ensure phases are followed (flexibly) ✓

**Important**: Every profile consumes methodology:
- Evaluator uses it to assess severity in context
- Recommender uses it to suggest appropriate techniques
- Builder uses it to determine acceptable exploitation depth
- Etc.

**Q10**: Should users be able to define custom methodologies?

**Answer**: **Yes, custom methodologies should be supported**

This aligns with the "modular" and "user-driven" design principles.

**Q11**: Can methodologies be combined/hybridized?

**Answer**: **Yes, methodology hybridization is allowed**

**Example**: "Red Team + Threat Model" combination
- Red Team's offensive techniques + Threat Model's systematic analysis
- Could be predefined combo or user-created on-the-fly

**Q12**: Can methodology evolve during an assessment based on findings?

**Answer**: **Yes, but requires user approval**

**Process**:
1. System detects findings that suggest methodology adjustment
2. Recommender suggests methodology evolution
3. User approves or rejects
4. If approved, methodology is updated for remainder of assessment

**Example**: 
"Architecture vulnerabilities detected → Recommend adding Threat Modeling phase to methodology"

**Q13**: How does methodology selection affect various aspects?

| Aspect | Effect |
|--------|--------|
| **Tool recommendations** | Based on: (1) OSS software available, (2) User-defined explicit preferences, (3) Web search recommendations (user-approved), (4) AI auto-figured (with user consent). Example: Threat Modeling doesn't need a web crawler |
| **Finding prioritization** | User input + profile guidance determine this initially |
| **Security Context/Map** | Only affects operating mode (Static/Ephemeral/Mixed). If Mixed mode restricts previous context from being used, methodology respects that. Output from methodology can enrich context if user approves |
| **Profile activation** | Falls more on Assessment Lifecycle than Methodology. All methodologies use approximately same lifecycle phases, which trigger profile activation |

---

### D. Cross-Cutting Concerns

**Q14**: Is there a fourth component we haven't explicitly named?

**Answer**: **No separate component - distributed across profiles**

**Rationale**: The suggested cross-cutting concerns are all valid requirements but are handled by/between profiles:

- **Context Manager** (decides data representation): 
  - Needed ✓
  - Handled by/between profiles based on their needs
  
- **State Machine** (tracks assessment progress):
  - Needed ✓  
  - Use cases depend on/handled by profiles
  - Orchestrator likely owns this
  
- **Decision Log** (audit trail of profile interactions):
  - Needed ✓
  - Use cases depend on/handled by profiles
  - Part of the logging system all profiles write to

**Implication**: No new component, but these are important capabilities to design into the profile interaction system.

---

## Decisions Log

### 2024-12-14 02:30 IST - CLI as Primary Interface
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

---

### 2024-12-14 02:45 IST - Profile Interaction Model
**Question**: How should the 9 AI profiles interact with each other?

**Options Considered**:
- Option A: Message Bus only (pub/sub events)
- Option B: Shared Context only (Security Context/Map)
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

---

### 2024-12-14 02:45 IST - Profile Activation Model
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

---

### 2024-12-14 02:45 IST - Conflict Resolution Hierarchy
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

---

### 2024-12-14 02:45 IST - Profile Memory/State
**Question**: Should profiles maintain their own memory and state?

**Options Considered**:
- Stateless profiles (fresh each invocation)
- Shared state only (Security Context/Map)
- Individual profile state + shared state

**Decision**: Profiles maintain individual state + shared state

**Rationale**:
- Essential for learning and improvement
- Prevents repetitive work (e.g., re-validating same finding)
- Enables personalization (e.g., learning user preferences)
- Supports skill progression tracking

**Implications**:
- Profile state stored in Security Context/Map
- State persistence depends on operating mode
- Need state serialization/deserialization

---

### 2024-12-14 02:45 IST - Methodology Configuration Format
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

---

### 2024-12-14 02:45 IST - Data Representation Conversion
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

---

### 2024-12-14 02:45 IST - Skills System Structure
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

---

## Implementation Roadmap

### Phase 1: Foundation
*Not started - awaiting design completion*

**Goals**:
- Core chassis architecture
- Internal data model (polymorphic representations)
- Basic tool adapters (starting with DAST)

**Deliverables**:
- Chassis framework
- Normalizer for common tools (Nuclei, ZAP)
- Basic object representation
- File-based storage

---

### Phase 2: Core Chassis & Profiles
*Not started - awaiting Phase 1*

**Goals**:
- CLI interface
- AI profile system (all 9 profiles)
- Approval system
- Security Context/Map (basic)

**Deliverables**:
- Working CLI with session management
- Profile interaction (message bus + shared context + orchestrator)
- User approval workflow
- Local database for context storage

---

### Phase 3: DAST Integration & Methodology
*Not started - awaiting Phase 2*

**Goals**:
- DAST tool adapters (comprehensive)
- Methodology framework
- Assessment lifecycle implementation
- Graph representation for attack surface

**Deliverables**:
- Support for 5+ DAST tools
- 3+ preset methodologies (Pentest, TM, Red Team)
- Custom methodology support
- Visual attack surface graph

---

### Phase 4: Learning System
*Not started - awaiting Phase 3*

**Goals**:
- DAG continuous learning
- Skills system (hierarchical tree)
- Active learning integration (CTF, labs)
- Passive learning (report parsing)

**Deliverables**:
- Skills database (400+ skills from GitHub)
- Skill progression tracking
- CTF integration
- Bug bounty report parser

---

### Phase 5: Experimental Features
*Not started - awaiting Phase 4*

**Goals**:
- SCA optimization (Symmetry/Chirality/Asymmetry)
- Token compression (rainbow tables)
- Advanced analytics
- Pattern recognition (emergence detection)

**Deliverables**:
- Compression algorithms
- Pattern detection engine
- Anomaly identification
- Markov chain analysis

---

### Phase 6: Electron UI/UX (Future)
*Planned for late-stage development*

**Goals**:
- Desktop GUI using Electron
- Visual workflow management
- Enhanced reporting/visualization
- Real-time assessment dashboard

**Timeline**: After core functionality + learning system, possibly during/after experimental features

**Deliverables**:
- Cross-platform desktop app
- Interactive attack surface visualization
- Drag-and-drop methodology builder
- Real-time profile activity monitor

---

## Future Considerations

### MCP Server Extension (Potential)
**Status**: Future consideration, not planned for initial phases

**Concept**: Extend SuBot as an MCP (Model Context Protocol) server

**Implications**:
- Would move to API-driven architecture
- Enables integration with other AI tools
- Allows external systems to leverage SuBot's capabilities
- Could expose profiles as separate MCP tools

**Decision**: Defer until core system is stable and proven

**Questions to Answer Later**:
- Which components should be exposed via MCP?
- How does this affect the local-first architecture?
- What security implications for exposing AI profiles as APIs?

---

## Experimental Features (Detailed)

### Optimization via SCA (Symmetry/Chirality/Asymmetry)
*Status: Concept exploration*

**Core Concepts**:
- **Chirality**: Left-handed vs right-handed patterns in data
- **Symmetry**: Identical patterns across different contexts
- **Asymmetry**: Unique patterns that break symmetry
- **Super-Symmetry**: Higher-order symmetries
- **Super-Asymmetry**: Higher-order asymmetries
- **Emergence**: Patterns that emerge from combination of simpler patterns
- **Markov Chain**: Probabilistic state transitions

**Potential Applications in AppSec**:

1. **Pattern Matching in Findings**:
   - Detect symmetric vulnerabilities across endpoints
   - Example: SQLi in /api/users AND /api/orders (symmetric pattern)
   - Asymmetric vulnerability suggests targeted attack or unique code path

2. **Token Compression**:
   - Identify redundant information in context
   - Rainbow table approach: hash common patterns → lookup instead of sending full text
   - Example: 100 similar findings → hash → single reference + delta

3. **Finding Relationships**:
   - Graph analysis using symmetry detection
   - Chirality identifies "mirror" attack paths
   - Example: Admin panel accessible via /admin AND /administrator (chiral paths)

4. **Noise Reduction**:
   - Eliminate redundant log entries
   - Deduplicate metadata
   - Compress repetitive scan results

5. **Emergence Detection**:
   - Identify attack patterns that emerge from individual findings
   - Example: 5 low-severity findings that together enable privilege escalation

**Questions to Explore**:
- Can we quantify symmetry in vulnerability patterns?
- How do we detect chirality in attack paths?
- What compression ratios are achievable?
- How does this integrate with the learning system?

---

## Notes & Insights

### From Initial Discussions

**Core Architectural Principles**:
- Profiles are NOT workflow stages - they are independent personas that critique and contribute
- Methodology informs approach, not rigid process
- Data model must be flexible, not one-size-fits-all
- Human approval is non-negotiable for all operations
- Local-first architecture for security and privacy
- CLI first, Electron UI much later (Phase 6)

**Skills System Insights** (from GitHub repo):
- Skills are concrete, actionable capabilities (not abstract concepts)
- Organized hierarchically: Fundamentals → Role-Specific → Domain-Specific
- 400+ skills defined across multiple security domains
- Progressive learning (prerequisites, leveling up)
- Different roles emphasize different skill sets
- Checkboxes suggest mastery tracking system

**Design Philosophy**:
- Deliberate before action (this document is proof)
- Use existing solutions (don't reinvent)
- Examples help understanding
- Track inconsistencies and incoherent details
- Diagrams for complex flows
- Reference documents for future lookup

---

## Revision History

### Version 0.3 - 2024-12-14 03:00 IST
**Changes**:
- Added Glossary section with all core concepts defined
- Integrated all answers from "My Insights.md"
- Added Skills system understanding from GitHub repo
- Expanded Design Questions & Decisions with full user answers
- Added examples throughout document
- Added detailed Experimental Features section
- Added Future Considerations section (MCP server)
- Fixed all date errors (2025 → 2024)
- Added revision history section
- Improved table formatting and structure

**Contributors**: User + Claude

---

### Version 0.2 - 2024-12-14 02:15 IST
**Changes**:
- Added CLI as primary interface decision
- Updated deployment model
- Added Electron UI to roadmap (Phase 6)
- Updated timestamp format

**Contributors**: User + Claude

---

### Version 0.1 - 2024-12-13 21:00 IST
**Changes**:
- Initial document creation
- Core components defined
- Open questions listed
- Basic structure established

**Contributors**: User + Claude

---

**Last Updated**: 2024-12-14 03:00 IST
**Document Version**: 0.3
**Status**: Active Design Phase - Answers Integrated
