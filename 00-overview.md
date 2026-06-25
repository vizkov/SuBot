# SuBot - Overview

**Last Updated:** 2025-12-15 18:37 IST
**Status:** Review

#overview

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

## Project Documents ^project-documents

### Core Documentation

```dataview
TABLE file.mtime as "Last Modified"
FROM ""
WHERE file.ext = "md" 
  AND !contains(file.path, "Archive") 
  AND !contains(file.path, "Misc")
  AND file.name != "Template"
SORT file.name ASC
```

### Component Designs

```dataview
TABLE file.mtime as "Last Modified"
FROM "2.1-component-designs" OR "2.2-security"
SORT file.folder ASC, file.name ASC
```

### Recent Activity

```dataview
TABLE file.mtime as "Last Modified"
FROM ""
WHERE file.ext = "md"
SORT file.mtime DESC
LIMIT 10
```

---

## Glossary ^glossary

### Core Concepts ^core-concepts

**Chassis**
The minimal core of the system that handles input → AI → output. All other functionality plugs into it. ^def-chassis

**Related:** [[01-architecture#^microkernel-architecture|Micro-kernel architecture]]

**Profiles**
Independent AI personas that act as critics and contributors, not sequential workflow stages. They operate concurrently on data/operations. ^def-profiles

**Related:** [[01-architecture#^ai-profile-system|AI Profile System]]

**Methodology**
Configuration that defines the approach, goals, and guidelines for a specific type of security assessment (e.g., Pentest vs Threat Modeling). ^def-methodology

**Related:** [[01-architecture#^methodology-framework|Methodology Framework]]

**Assessment Lifecycle**
Framework of phases (Purpose, Requirement, Scope, etc.) that guide but don't rigidly constrain the assessment process. ^def-assessment-lifecycle

**Related:** [[01-architecture#^assessment-lifecycle|Assessment Lifecycle]]

**Security-Context Map**
Persistent or ephemeral knowledge base that:
- **Enriches input**: Adds historical context, patterns, and learned knowledge to incoming data
- **Is enriched by output**: Stores new findings, decisions, and outcomes for future reference
- Maintains target fingerprints, vulnerability patterns, and skill mastery data
- Operates in Static, Ephemeral, or Mixed modes

^def-security-context-map

**Related:** [[01-architecture#^security-context-map-section|Security-Context Map details]]

**DAG Continuous Learning**
Directed Acyclic Graph representing skill dependencies and progression, inspired by character progression systems in games. ^def-dag-cl

**Related:** [[01-architecture#^dag-continuous-learning|DAG Continuous Learning]]

**Skills**
Concrete, actionable capabilities (e.g., "SQL Injection Tester", "Firewall Rule Automator") organized hierarchically and progressively. ^def-skills

**Related:** [[01-architecture#^dag-continuous-learning|Skills System]]

**Tool Adapter**
Plugin interface that connects external security tools to the system. Responsible for invoking tools, capturing output, and passing raw data to the Normalizer. Think of it as the "driver" for each tool. ^def-tool-adapter

**Related:** [[01-architecture#^tool-adapter-system|Tool Adapter System]]

**Normalizer**
Component that parses raw tool outputs and transforms them into standardized internal data structures (Finding/Evidence objects). Handles diverse formats (JSON, XML, text) and malformed data. Think of it as the "translator" between tools and the system. ^def-normalizer

**Polymorphic Data Model**
Context-dependent data representation (object/graph/compressed/DAG) that adapts based on operational needs. ^def-polymorphic-data-model

**Related:** [[01-architecture#^internal-data-model|Internal Data Model]]

### AI Profiles ^ai-profiles

**Evaluator**
Assesses the validity and severity of actions/findings from other profiles. ^def-evaluator

**Validator**
Independently verifies correctness of outputs from other profiles. ^def-validator

**Enforcer**
Ensures compliance with methodology rules, standards, and constraints. ^def-enforcer

**Recommender**
Suggests next actions, techniques, or tools to other profiles. ^def-recommender

**Orchestrator**
Manages workflows, action queues, and message passing between profiles. ^def-orchestrator

**Builder**
Constructs payloads, test cases, and exploit code based on validated findings. ^def-builder

**Debugger**
Diagnoses tool failures, scan errors, and operational issues. ^def-debugger

**Synthesizer**
Aggregates findings into coherent security narratives and reports. ^def-synthesizer

**Optimizer**
Identifies and eliminates redundancy, reduces token usage, streamlines operations. ^def-optimizer

**Related:** [[01-architecture#^ai-profile-system|AI Profile System details]]

### Operating Modes ^operating-modes

**Static Mode (Pro-context)**
Full persistence of learning, findings, and context across sessions. ^def-static-mode

**Ephemeral Mode (Blind)**
No persistence; each session starts fresh with no historical context. ^def-ephemeral-mode

**Mixed Mode**
Configurable selective persistence (e.g., keep methodology learnings but not target-specific data). ^def-mixed-mode

**Related:** [[01-architecture#^security-context-map-section|Security-Context Map operating modes]]

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

---

## Revision History

### Version 0.5 - 2025-12-14 11:30 IST
**Changes**:
- Changed "Security Context/Map" to "Security-Context Map" throughout all modules
- Improved Tool Adapter and Normalizer definitions (clearer distinction)
- Removed "Deliberate Before Action" design principle (was instruction, not principle)
- Fixed Internal Data Model Q1 & Q2 - marked as PENDING DECISION
- Added data flow description for Security-Context Map (enriches input, enriched by output)
- Added DAG CL relationship with Security-Context Map in 01-architecture.md
- Fixed all dates from 2024 to 2025
- Applied corrections across modular structure

**Contributors**: User + Claude
**Files Modified**: 00, 02, 03, 04

---

### Version 0.3 - 2025-12-14 03:00 IST
**Changes**:
- Added Glossary section with all core concepts defined
- Integrated all answers from "My Insights.md"
- Added Skills system understanding from GitHub repo
- Expanded Design Questions & Decisions with full user answers
- Added examples throughout document
- Added detailed Experimental Features section
- Added Future Considerations section (MCP server)
- Attempted date correction (incomplete - fixed properly in v0.5)
- Added revision history section
- Improved table formatting and structure

**Contributors**: User + Claude

---

### Version 0.2 - 2025-12-14 02:15 IST
**Changes**:
- Added CLI as primary interface decision
- Updated deployment model
- Added Electron UI to roadmap (Phase 6)
- Updated timestamp format

**Contributors**: User + Claude

---

### Version 0.1 - 2025-12-13 21:00 IST
**Changes**:
- Initial document creation
- Core components defined
- Open questions listed
- Basic structure established

**Contributors**: User + Claude
