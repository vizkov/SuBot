# SuBot - Implementation Roadmap

**Last Updated:** 2025-12-15 18:37 IST
**Status:** Review

#roadmap

---

## Implementation Roadmap ^roadmap

> [!important] Design First
> All phases require complete design before implementation begins.

### Phase 1: Foundation ^phase-1
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

**Related:** [[01-architecture#^microkernel-architecture|Micro-kernel architecture]], [[2.1-component-designs/2.1.1-chassis|Chassis design]]

**Implementation Tasks (Chassis)**:
1. [ ] Implement core Chassis class
2. [ ] Implement Plugin interface
3. [ ] Implement MessageBus
4. [ ] Implement PluginRegistry
5. [ ] Create sample plugin (Orchestrator)
6. [ ] Test boot sequence
7. [ ] Document plugin development guide

---

### Phase 2: Core Chassis & Profiles ^phase-2
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

**Related:** [[01-architecture#^ai-profile-system|AI Profile System]], [[01-architecture#^cli-interface|CLI Interface]]

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
