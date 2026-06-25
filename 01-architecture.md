# SuBot - System Architecture

**Last Updated:** 2025-12-15 15:12 IST
**Status:** Review

#architecture #component

---

## System Architecture ^system-architecture

### High-Level Data Flow ^high-level-data-flow
```
Input (any tool/format) 
  ↓
[Normalizer] - converts to internal format
  ↓
[Security-Context Map] - enriches with history/knowledge
  ↓
[AI Chassis with active profile(s)] - reasons about it
  ↓
[Action/Recommendation] - with justification/logs
  ↓
[User Approval Gate]
  ↓
[Execution/Storage] - updates context, triggers next cycle
```

---

## Core Components ^core-components

### 1. CLI Interface & UX ^cli-interface
*Status: Design in progress*

**Current Focus**: Command-line interface
**Future Expansion**: Electron-based UI/UX (planned for late-stage development, possibly during/after experimental features)

**Pending Design Decisions:**
- Command vs conversational vs hybrid model
- Session management approach
- Approval UX pattern

> [!info] Related Decisions
> See [[03-decisions-log#^adr-cli-primary-interface|CLI interface decision]] for rationale.

---

### 2. Micro-kernel Architecture ^microkernel-architecture
*Status: Concept defined*

**Design**: 
- Minimal [[00-glossary#^def-chassis|Chassis]] handles: input → AI → output
- All other functionality via plugins/modules
- Configuration-driven behavior

> [!important] Core Design Principle
> This architecture enables the tool-agnostic design from [[00-overview#Core Design Principles]].

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

---

### 3. AI Profile System (9 Personas) ^ai-profile-system
*Status: Design in progress*

**Profiles are independent personas, not workflow stages.**

**Key Design Decision**: Profiles operate concurrently as critics and contributors, not sequentially.

> [!important] Interaction Model Decision
> See [[03-decisions-log#^adr-profile-interaction|profile interaction decision]] for implementing all three mechanisms.

#component/profile

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

**Interaction Model**: ^profile-interaction-model
All three interaction mechanisms serve distinct purposes:
- **Message Bus**: Maintains workflow order and event-driven activation
- **Shared Context**: Profiles read/write to [[00-glossary#^def-security-context-map|Security-Context Map]] directly
- **Orchestrator-Mediated**: Orchestrator manages queuing and prioritization

**Related**: [[02-design-questions#B. AI Profile Interaction Model]]

**Activation Model**: ^profile-activation-model
Event-triggered activation - specific events trigger specific profiles:
- More efficient than always-on operation
- Examples:
  - New finding → Evaluator + Validator
  - Scan failure → Debugger
  - User asks "what next?" → Recommender

**Decision**: [[03-decisions-log#^adr-profile-activation|Profile activation decision]]

**Conflict Resolution**: ^profile-conflict-resolution
Weighted voting with clear hierarchy:
- **User >>> Enforcer > Orchestrator**
- Certain things get vetoed automatically by Enforcer
- User has final say on all conflicts

> [!warning] Human-in-the-Loop Requirement
> User always has ultimate authority - profiles cannot override human decisions.

**Decision**: [[03-decisions-log#^adr-conflict-resolution|Conflict resolution decision]]

**State/Memory**: ^profile-state-memory
Profiles maintain their own memory/state:
- Validator remembers what it's already validated
- Recommender tracks accepted/rejected recommendations
- Optimizer learns which optimizations worked
- This is vital for avoiding repetition and improving over time

**Decision**: [[03-decisions-log#^adr-profile-memory|Profile memory decision]]

---

### 4. Methodology Framework ^methodology-framework
*Status: Design decisions made*

#component

**Methodologies supported:** ^supported-methodologies
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

**Related**: [[02-design-questions#C. Methodology Framework]]

**Methodology Purpose**: ^methodology-purpose
- Define approach and goals for security review
- Inform which techniques/tools are appropriate
- **Do NOT** affect profile behavior (profiles are "rails" for the system)
- Every [[00-glossary#^def-ai-profile|profile]] consumes [[00-glossary#^def-methodology|methodology]] as input and acts on it differently

> [!important] Profile Independence
> Methodologies don't dictate which profiles activate - that's determined by events and the [[#^profile-activation-model|activation model]].

**Configuration Format**: ^methodology-config-format
YAML with flexible base structure that can be expanded/customized

**Decision**: [[03-decisions-log#^adr-methodology-config|Methodology config decision]]

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

**Enforcer Profile Usage**:
Enforcer ensures compliance with methodology:
- Checks action compliance with guidelines
- Verifies goals are being addressed
- Ensures phases are followed (flexibly, not strictly)
- **Note**: All profiles consume methodology, not just Enforcer

**Custom Methodologies**:
Users can define custom methodologies

**Methodology Combination**:
Methodologies can be hybridized (e.g., "Red Team + Threat Model")

**Methodology Evolution**:
Can be suggested to user based on findings, then approved
- Example: "Based on architecture findings, recommend adding Threat Modeling phase"

**Methodology Effects**:

| Aspect | How Methodology Affects It |
|--------|---------------------------|
| **Tool recommendations** | Based on OSS availability, user-defined preferences, or web search (with approval) |
| **Finding prioritization** | User input + profile guidance initially |
| **Security-Context Map** | Only affects operating mode (Static/Ephemeral/Mixed). Doesn't dictate, but can be enriched by methodology output if user approves |
| **Profile activation** | Falls more on Assessment Lifecycle than Methodology. All methodologies use approximately same lifecycle phases |

---

### 5. Assessment Lifecycle ^assessment-lifecycle
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

> [!info] Flexible Framework
> Assessment can loop back between phases as new information emerges (e.g., Triage reveals new Scope requirements).

**Example Flow**:
```
User starts assessment → Purpose (define goals) → Scope (target boundaries)
→ Restrictions (what's off-limits) → Pre-requisites (access, credentials)
→ Scan (execute tools) → Triage (categorize findings)
→ QA/QC (validate, prioritize) → Observations (document, report)
```

Assessment can loop back (e.g., Triage reveals new Scope requirements).

---

### 6. Security-Context Map ^security-context-map-section
*Status: Concept defined*

#component #security

**Operating Modes:**
- Static (Pro-context mode) - persistent learning
- Ephemeral (Blind-mode) - no persistence
- Mixed-configurable (Pick and choose)

**Core Functions:**
- Enriches profile knowledge
- Enriches methodology application
- Enriches DAG continuous learning
- Can generate/use metadata tags

**Data Flow**:
- **Enriches Input**: When data enters the system, Security-Context Map adds historical context, known patterns, previous learnings, and relevant metadata
- **Enriched by Output**: When profiles produce results, decisions, or findings, these are stored back into Security-Context Map, continuously building institutional knowledge

**Storage**: Local (files/database)

**Example Data Stored**:
```json
{
  "target": "api.example.com",
  "tech_stack": ["Python", "Flask", "PostgreSQL"],
  "historical_findings": [
    {"type": "SQLi", "endpoint": "/api/login", "severity": "high", "date": "2025-12-01"}
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

---

### 7. Tool Adapter System ^tool-adapter-system
*Status: Concept defined*

#component/adapter

**Design**: Generic adapters for any tool output, normalize to internal data format(s)

**Starting focus**: DAST tools (Nuclei, ZAP, Burp)

> [!info] Tool-Agnostic Design
> This enables the core principle of accepting ANY tool output - see [[00-overview#1. Modularity = Tool Agnostic]].

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

---

### 8. Internal Data Model ^internal-data-model
*Status: Design in progress*

#component

**Key Insight**: Data model is polymorphic, context-dependent

> [!question] Open Design Questions
> See [[02-design-questions#A. Internal Data Model]] for pending decisions on representation selection and conversion logic.

**Representation Types**:
- Finding from tool → **Object** representation
- Attack surface mapping → **Graph** representation
- Token optimization → **Compressed** representation
- Skill progression → **DAG** representation

**Representation Decision Logic**:
- Pending design discussion (see 03-design-questions.md Q1)

**Conversion Between Representations**:
- Pending design discussion (see 03-design-questions.md Q2)

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

---

### 9. DAG Continuous Learning ^dag-continuous-learning
*Status: Framework defined, Skills system clarified*

#component

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

**Relationship with Security-Context Map**:
- DAG CL **informs** Security-Context Map by providing skill mastery data, learning progress, and capability assessments
- DAG CL **enriches** Security-Context Map with identified patterns, successful techniques, and skill gap analysis
- Security-Context Map stores DAG CL state and progression data
- Bidirectional relationship: learnings flow both ways

**Example Skill Progression**:
```yaml
skill: "SQL Injection Detection"
category: "Web Application Security"
prerequisites: 
  - "HTTP Fundamentals"
  - "Database Basics"
current_level: 72/100
experience_sources:
  - ctf_challenge: "SQLi Lab 5" (2025-11-15) → +8 XP
  - bug_bounty_report: "Uber SQLi Analysis" (2025-12-01) → +5 XP
  - assessment: "Target ABC SQLi found" (2025-12-10) → +10 XP
next_level_requirements:
  - Complete 3 more CTF challenges OR
  - Successfully identify SQLi in 5 live assessments
```

---

### 10. Hybrid LLM Strategy ^hybrid-llm-strategy
*Status: Concept defined*

#component

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

---

### 11. Approval System ^approval-system
*Status: Requirements defined*

#component

**Requirement**: Human approval required for ALL operations

> [!important] Core Requirement
> This is a fundamental principle - see [[00-overview#Core Design Principles]].

**Pending**: Pre-approval vs step-by-step vs trust levels

**Related**: [[02-design-questions#E. Chassis Component Design]]
