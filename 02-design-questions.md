# SuBot - Design Questions & Decisions

**Last Updated:** 2025-12-15 18:37 IST
**Status:** Review

#question

---

## Design Questions & Decisions ^design-questions

### A. Internal Data Model ^internal-data-model-questions

**Q1**: How does the system decide which data representation to use (object/graph/compressed/DAG)?
^question-data-representation

**Status**: **PENDING DECISION** - To be discussed and decided during component design

**Options to Explore**:
- Profile-driven? (Each profile requests format it needs)
- Methodology-driven? (Methodology suggests optimal representation)
- Dynamic based on operation? (AI/Agentic patterns decide in real-time)
- Hybrid approach?

**Context from Initial Discussion**:
"This depends on whether we use AI/Agentic vs conventional approach. If AI-driven, relies on AI/RAG patterns. If conventional, explicit decisions for each use-case. Component-specific design will clarify this further."

**Note**: Flagged as "to be discussed further" - not answered yet

**Related:** [[01-architecture#^internal-data-model|Internal Data Model]]

---

**Q2**: Do these representations need to convert between each other?
^question-data-conversion

**Status**: **PENDING DECISION** - To be discussed and decided during component design

**Considerations**:
- Can a finding object become a node in the attack graph?
- Can graph patterns collapse into compressed tokens for the LLM?
- What are the conversion costs and complexity?
- Is bidirectional conversion needed or one-way?
- How to handle data loss during compression?

**Initial Thought**:
"Conversions seem beneficial - enables information exchange and multi-view analysis. But implementation details need to be worked out."

**Note**: Conversion capability seems beneficial but specifics need component design discussion

**Decision**: [[03-decisions-log#^adr-data-conversion|Data conversion decision]]

---

### B. AI Profile Interaction Model ^profile-interaction-questions

**Q3**: How do profiles interact with each other?
^question-profile-interaction

**Answer**: **All 3 options serve a purpose - implement all**
- **Option A - Message Bus**: Maintains workflows and order (pub/sub events)
- **Option B - Shared Context**: Directly contributes to the system's feature/goal (all read/write to Security-Context Map)
- **Option C - Orchestrator-Mediated**: Definitely the Orchestrator's job (centralized hub)

**Rationale**: Each serves different aspects:
- Message Bus for event-driven triggers
- Shared Context for knowledge accumulation
- Orchestrator for workflow management

**Decision**: [[03-decisions-log#^adr-profile-interaction|Profile interaction decision]]

---

**Q4**: When does each profile activate?
^question-profile-activation

**Answer**: **Option B - Triggered (events trigger specific profiles)**

**Rationale**: Optimal approach balancing efficiency and coverage
- New finding → Evaluator + Validator
- Scan failure → Debugger  
- User query "what next?" → Recommender
- Not always-on (too expensive), not user-selectable (defeats profiles as "rails")

**Decision**: [[03-decisions-log#^adr-profile-activation|Profile activation decision]]

---

**Q5**: What happens when profiles disagree?
^question-profile-conflict

**Answer**: **Weighted vote with veto hierarchy**

**Decision Hierarchy**: **User >>> Enforcer > Orchestrator**
- User has ultimate authority
- Enforcer can veto on methodology/compliance violations
- Orchestrator mediates between other profiles
- Certain rules get automatic veto (e.g., Enforcer blocks critical findings without validation)

**Decision**: [[03-decisions-log#^adr-conflict-resolution|Conflict resolution decision]]

**Example Scenario**:
```
Evaluator: "This is high severity"
Validator: "I can't reproduce, likely false positive"  
Enforcer: "Methodology requires 3 confirmations for high severity"
→ Orchestrator: "Requesting additional validation"
→ If still no consensus → Escalate to User
```

---

**Q6**: Do profiles maintain their own memory/state?
^question-profile-memory

**Answer**: **Yes, vital for system effectiveness**

Specific behaviors:
- **Validator** remembers what it's already validated (avoid re-validation)
- **Recommender** tracks accepted/rejected recommendations (learn preferences)
- **Optimizer** learns which optimizations worked (improve over time)
- **Evaluator** tracks false positive patterns
- **Builder** remembers successful payload patterns

**Implications**:
- Each profile needs persistent storage for its state
- State should be part of [[00-overview#^def-security-context-map|Security-Context Map]]
- State persists based on operating mode (Static/Ephemeral/Mixed)

**Decision**: [[03-decisions-log#^adr-profile-memory|Profile memory decision]]

---

### C. Methodology Framework ^methodology-questions

**Q7**: How is a methodology specified/structured?
^question-methodology-format

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

**Decision**: [[03-decisions-log#^adr-methodology-config|Methodology config decision]]

---

**Q8**: Should methodologies define which profiles are emphasized?
^question-methodology-profiles

**Answer**: **No, profiles are not affected by methodology**

**Rationale**: Profiles operate on/with each other's output. They act like "rails" for the system.
- Profiles do not vary based on input
- All profiles consume methodology as input
- Each profile acts on methodology differently
- Methodology doesn't dictate which profiles are active

---

**Q9**: How does the Enforcer profile use methodology?
^question-enforcer-methodology

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

---

**Q10**: Should users be able to define custom methodologies?

**Answer**: **Yes, custom methodologies should be supported**

This aligns with the "modular" and "user-driven" design principles.

---

**Q11**: Can methodologies be combined/hybridized?

**Answer**: **Yes, methodology hybridization is allowed**

**Example**: "Red Team + Threat Model" combination
- Red Team's offensive techniques + Threat Model's systematic analysis
- Could be predefined combo or user-created on-the-fly

---

**Q12**: Can methodology evolve during an assessment based on findings?

**Answer**: **Yes, but requires user approval**

**Process**:
1. System detects findings that suggest methodology adjustment
2. Recommender suggests methodology evolution
3. User approves or rejects
4. If approved, methodology is updated for remainder of assessment

**Example**: 
"Architecture vulnerabilities detected → Recommend adding Threat Modeling phase to methodology"

---

**Q13**: How does methodology selection affect various aspects?

| Aspect | Effect |
|--------|--------|
| **Tool recommendations** | Based on: (1) OSS software available, (2) User-defined explicit preferences, (3) Web search recommendations (user-approved), (4) AI auto-figured (with user consent). Example: Threat Modeling doesn't need a web crawler |
| **Finding prioritization** | User input + profile guidance determine this initially |
| **Security-Context Map** | Only affects operating mode (Static/Ephemeral/Mixed). If Mixed mode restricts previous context from being used, methodology respects that. Output from methodology can enrich context if user approves |
| **Profile activation** | Falls more on Assessment Lifecycle than Methodology. All methodologies use approximately same lifecycle phases, which trigger profile activation |

---

### D. Cross-Cutting Concerns ^cross-cutting-questions

**Q14**: Is there a fourth component we haven't explicitly named?
^question-fourth-component

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

### E. Chassis Component Design ^chassis-questions

**Q15**: Async vs Sync Message Bus?
^question-async-sync

**Options**:
- Async for better performance
- But complicates debugging
- Start with sync, add async later?

**Status**: Open question - needs decision

---

**Q16**: Plugin Hot-Reload?
^question-plugin-hot-reload

**Options**:
- Reload plugins without system restart
- Useful for development
- Add in Phase 2?

**Status**: Open question - needs decision

---

**Q17**: Plugin Dependencies?
^question-plugin-dependencies

**Questions**:
- Can plugins depend on other plugins?
- How to handle load order?

**Status**: Open question - needs decision

---

**Q18**: Plugin Versioning?
^question-plugin-versioning

**Questions**:
- How to handle plugin API changes?
- Version compatibility checks?

**Status**: Open question - needs decision
