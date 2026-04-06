# Changelog

This project follows [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Changed

- Translated all skill files from Korean to English: `SKILL.md`, all 6 reference documents, `CHANGELOG.md`, and plugin metadata (`plugin.json`, `marketplace.json`)
- Added `CLAUDE.md` to the repository root with plain-English installation and usage documentation

## [1.1.0] - 2026-04-05

### Added

- **Phase 0: Current State Audit** — On trigger, first checks the existing harness state and routes to one of three tracks: new build / extend existing / operate & maintain
- **Extend Existing Phase Selection Matrix** — Decision table specifying which Phases are required for each scenario: add agent / add skill / change architecture
- **Phase 3/4 CLAUDE.md Interim Sync** — Immediately reflects newly created agents and skills into CLAUDE.md right after creation (resilient to mid-session interruptions)
- **Phase 5-4: CLAUDE.md Harness Context Registration** — Records agent team structure, skill list, execution rules, directory structure, and change history. Includes a responsibility split table for CLAUDE.md vs. Orchestrator
- **Phase 5-5: Follow-up Task Support** — Orchestrator description must include follow-up keywords; Phase 0 context check step auto-identifies initial / partial re-run / new run
- **Phase 5 Orchestrator Edit Path** — Guide for modifying the existing Orchestrator rather than creating a new one when extending
- **Phase 7: Harness Evolution Mechanism** — Collect feedback after execution → map feedback types to fix targets → record change history → automated evolution trigger
- **Phase 7-5: Operations/Maintenance Workflow** — 4-step process: current state audit → incremental fixes → CLAUDE.md sync → change verification
- **Operations/maintenance triggers in description** — Keywords: 'harness check', 'harness audit', 'harness status', 'agent/skill sync'
- **Strengthened output checklist** — Added: CLAUDE.md sync complete, change history recorded, Phase 0 context check items
- Added Phase 0 (context check) to Orchestrator template — applies to both Agent Teams and Subagent modes
- Follow-up task keyword patterns included in Orchestrator description template

### Changed

- Core principles expanded from 2 to 4 (CLAUDE.md registration, evolution system added)
- **"Evolution log" → "Change history" unified** — Name and schema (4 columns: date / change / target / reason) consolidated across all sections
- **Phase 1 Step 3** — Updated to perform conflict analysis based on Phase 0 audit results (removes duplication)
- **5-4 CLAUDE.md template code block** — Fixed nested rendering breakage (3 backticks → 4 backticks)
- **Responsibility split table expanded** — Added rows for skill list, directory structure, change history
- **Orchestrator template** — Added Phase 0 context check step and follow-up task keyword guide

## [1.0.1] - 2026-03-28

### Changed

- Removed duplicate content between SKILL.md and references (330 lines → 285 lines)
  - Phase 2-1: Execution mode comparison table/bullets → core principles + agent-design-patterns.md pointer
  - Phase 2-3: Agent separation criteria bullets → 4-axis summary + agent-design-patterns.md pointer
  - Phase 3: Agent definition template code block → required sections list + references pointer
  - Phase 5-2: Error handling 5-row table → core principles + orchestrator-template.md pointer

## [1.0.0] - 2026-03-27

### Added

- Harness configuration meta-skill based on a 6-Phase workflow
- 6 agent architecture patterns (Pipeline, Fan-out/Fan-in, Expert Pool, Generate-Verify, Supervisor, Hierarchical Delegation)
- Agent Teams / Subagent execution mode support
- Progressive Disclosure-based skill creation guide
- Orchestrator templates (Agent Teams mode + Subagent mode)
- QA agent integration guide (based on 7 real-project bug cases)
- Skill testing/evaluation methodology (With-skill vs Without-skill comparison)
- 5 real-world team composition examples (research, novel, webtoon, code review, migration)
