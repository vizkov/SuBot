# SuBot 
Modular, AppSec bot 🤖

I basically wanted to learn about AI prompting ⌨️, and I decided to do that by designing an AppSec bot. None of this was ever executed or implemented on any scale. The core idea was to use AI as a co-creator/architect.

Although it started as a project with a particular [vision/goal](00-overview.md) ✨, it ended up becoming more of a playground for different tangents and concepts that were designed simultaneously 🛝.  

Note: A lot has changed since I carried out this elaborate activity/exercise (2025), and it may be outdated.

## What the design does 🧭

SuBot was designed as an AI "AppSec engineer": a command-line tool (with an Electron UI sketched for later) that would start with DAST (dynamic testing), keep everything on the local machine, and require a human to approve every operation. It was only ever a design: nothing was built or run.

It would not run scanners itself. Output from any security tool (Nuclei, Burp, ZAP, custom scripts) would be fed in and converted to one standard internal format, and AI would then reason about it:

```
Tool output → Normalizer → Security-Context Map (history/knowledge) → AI profiles → Recommendation (with reasons) → You approve → Execute / store
```

The main ideas:

- **Chassis (micro-kernel):** a minimal core that loads plugins, routes messages between them and manages their lifecycle. Everything else (AI profiles, tool adapters, storage, UI) was to be a plugin, and methodologies (pentest, threat modeling, ...) were to be YAML configs rather than code. → [Architecture](01-architecture.md), [Chassis design](1.1-component-designs/1.1.1-chassis.md)
- **Nine AI profiles:** independent personas that act as critics and contributors rather than workflow stages: Evaluator, Validator, Enforcer, Recommender, Orchestrator, Builder, Debugger, Synthesizer and Optimizer. Events would trigger them, they would talk over a message bus, a shared context and the Orchestrator, and conflicts would be settled User >>> Enforcer > Orchestrator.
- **Security-Context Map:** a knowledge base that adds history and learned patterns to incoming data and is updated by the results. It could be persistent (static), blank every session (ephemeral) or mixed.
- **DAG continuous learning:** skills (e.g. "SQL injection tester") sit in a prerequisite tree, like character progression in a game, and would be levelled up through CTFs, labs and parsed bug bounty reports. The [`Skills/`](Skills) folder holds the role-specific skill lists (Pentester, AI Pentester, Product Security, Security Architect, ...).
- **Hybrid LLMs:** a cheaper open-source model for the bulk work (processing context, spotting patterns) and a paid model only for hard reasoning, so fewer tokens would go to the paid one.
- **Self-security:** [a threat-focused review of the bot itself](1.2-security/security-considerations.md): plugin validation, isolation, approval-bypass prevention, command-injection and output validation, and an audit trail.

Experimental ideas (token compression, symmetry/asymmetry-based optimisation) and a possible MCP-server mode were parked in [Future considerations](05-future-considerations.md).

## Where things are 🗂️

| File | What it holds |
|---|---|
| [00-overview.md](00-overview.md) | Vision, glossary, design principles |
| [01-architecture.md](01-architecture.md) | Components and data flow |
| [02-design-questions.md](02-design-questions.md) | Open questions and the answers reached |
| [03-decisions-log.md](03-decisions-log.md) | Decisions with options and rationale |
| [04-roadmap.md](04-roadmap.md) | A six-phase outline that was never started: foundation → chassis and profiles → DAST and methodology → learning → experimental → Electron UI |
| [05-future-considerations.md](05-future-considerations.md) | Ideas that were parked |
| [Skills/](Skills) | Skill trees (several were left as empty stubs) |
| [Misc/](Misc) | The instructions I gave Claude, plus older drafts in `Archive/` |

How I worked with Claude on this:
- [Claude's Project Instructions](https://github.com/vizkov/SuBot/blob/main/Misc/Bootstrap-Instructions.md)
- [Obsidian ref for Claude to follow](https://github.com/vizkov/SuBot/blob/main/Misc/Obsidian-Guide.md)
- [Ref guide for Claude to follow as an architect](https://github.com/vizkov/SuBot/blob/main/Misc/Project-Instructions.md)
- [Ref guide for Claude to follow for testing and validating its output](https://github.com/vizkov/SuBot/blob/main/Misc/Testing-Guide.md)

The notes were written in [Obsidian](https://obsidian.md) (`[[links]]`, Dataview blocks), so they read best there.
