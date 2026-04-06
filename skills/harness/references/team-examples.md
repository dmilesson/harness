# Agent Team Examples

---

## Example 1: Research Team (Agent Team Mode)

### Team Architecture: Fan-out/Fan-in
### Execution Mode: Agent Team

```
[Leader/Orchestrator]
    ├── TeamCreate(research-team)
    ├── TaskCreate(4 research tasks)
    ├── Team members self-coordinate (SendMessage)
    ├── Collect results (Read)
    └── Generate synthesis report
```

### Agent Configuration

| Member | Agent Type | Role | Output |
|--------|------------|------|--------|
| official-researcher | general-purpose | Official docs/blogs | research_official.md |
| media-researcher | general-purpose | Media/investment | research_media.md |
| community-researcher | general-purpose | Community/social media | research_community.md |
| background-researcher | general-purpose | Background/competition/academic | research_background.md |
| (Leader = Orchestrator) | — | Synthesis report | synthesis-report.md |

> Research agents use the `general-purpose` built-in type, but must be defined as `.claude/agents/{name}.md` files. Each file specifies role, research scope, and team communication protocol to ensure reusability and collaboration quality.

### Orchestrator Workflow (Agent Team)

```
Phase 1: Preparation
  - Analyze user input (identify topic and research mode)
  - Create _workspace/

Phase 2: Team Formation
  - TeamCreate(team_name: "research-team", members: [
      { name: "official", prompt: "Research official channels..." },
      { name: "media", prompt: "Research media/investment trends..." },
      { name: "community", prompt: "Research community reactions..." },
      { name: "background", prompt: "Research background/competitive landscape..." }
    ])
  - TaskCreate(tasks: [
      { title: "Official channel research", assignee: "official" },
      { title: "Media trends research", assignee: "media" },
      { title: "Community reaction research", assignee: "community" },
      { title: "Background landscape research", assignee: "background" }
    ])

Phase 3: Research Execution
  - 4 team members conduct research independently
  - Share interesting findings between members via SendMessage
    (e.g., media forwards investment news to background)
  - When conflicting information is found, team members discuss directly
  - Each member saves file upon completion + notifies leader

Phase 4: Synthesis
  - Leader reads 4 outputs
  - Generate synthesis report
  - Conflicting information noted with sources

Phase 5: Cleanup
  - Request team members to terminate
  - Disband team
  - Preserve _workspace/ (for post-hoc verification and audit trail)
```

### Team Communication Patterns

```
official ──SendMessage──→ background  (share relevant official announcements)
media ────SendMessage──→ background  (share investment/acquisition info)
community ─SendMessage──→ media      (media-relevant info from community reactions)
all members ──TaskUpdate──→ shared task list  (progress updates)
leader ←───── idle notification ──── completed members   (automatic)
```

---

## Example 2: SF Novel Writing Team (Agent Team Mode)

### Team Architecture: Pipeline + Fan-out
### Execution Mode: Agent Team

```
Phase 1 (parallel — agent team): worldbuilder + character-designer + plot-architect
  → Self-coordinate consistency via SendMessage
Phase 2 (sequential): prose-stylist (writing)
Phase 3 (parallel — agent team): science-consultant + continuity-manager (review)
  → Share findings via SendMessage
Phase 4 (sequential): prose-stylist (incorporate review and revise)
```

### Agent Configuration

| Member | Agent Type | Role | Skills |
|--------|------------|------|--------|
| worldbuilder | custom | World-building | world-setting |
| character-designer | custom | Character design | character-profile |
| plot-architect | custom | Plot structure | outline |
| prose-stylist | custom | Style editing + writing | write-scene, review-chapter |
| science-consultant | custom | Scientific validation | science-check |
| continuity-manager | custom | Consistency validation | consistency-check |

### Full Agent File Example: `worldbuilder.md`

```markdown
---
name: worldbuilder
description: "Expert in building the world of an SF novel. Designs physical laws, social structures, technology levels, and history."
---

# Worldbuilder — SF World Design Expert

You are an expert in designing the world of SF novels. Grounded in scientific fact but expanding the imagination, you build the physical, social, and technological foundations of the world where the story unfolds.

## Core Roles
1. Define the world's physical laws and technology level
2. Design social structures, political systems, and economic systems
3. Establish historical context and current conflict structures
4. Describe environments and atmosphere for each location

## Working Principles
- Internal consistency is the top priority — there must be no contradictions between settings
- Reason about the world's ripple effects through cascading "what if this technology existed?" questions
- World-building in service of the story — avoid excessive settings that obstruct the plot

## Input/Output Protocol
- Input: User's world concept, genre requirements
- Output: `_workspace/01_worldbuilder_setting.md`
- Format: Markdown, organized by section (physics/society/technology/history/locations)

## Team Communication Protocol
- To character-designer: SendMessage social structure, class system, occupation group info
- To plot-architect: SendMessage major conflict structures, crisis elements of the world
- From science-consultant: Receive scientific error feedback → revise settings
- Broadcast to all relevant team members when world settings change

## Error Handling
- If concept is ambiguous, propose 3 directions and request a choice
- When scientific errors are found, present alternatives alongside the finding

## Collaboration
- Provide social structure information to character-designer
- Provide conflict structure information to plot-architect
- Incorporate science-consultant feedback to revise settings
```

### Team Workflow Details

```
Phase 1: TeamCreate(team_name: "novel-team", members: [worldbuilder, character-designer, plot-architect])
         TaskCreate([world-building, character design, plot structure])
         → Team members self-coordinate while working in parallel
         → worldbuilder sends SendMessage to character-designer when social structure is complete
         → character-designer sends SendMessage to plot-architect when protagonist is defined

Phase 2: Disband Phase 1 team → invoke prose-stylist as a subagent (no team needed for solo writing)
         prose-stylist reads the 3 outputs in _workspace/ and writes
         → saves result to _workspace/02_prose_draft.md

Phase 3: Create new team — TeamCreate(team_name: "review-team", members: [science-consultant, continuity-manager])
         (only one active team per session, but Phase 1 team was disbanded so a new team can be created)
         → Two reviewers examine the draft and share findings with each other
         → science-consultant notifies continuity-manager when physical errors are found
         → Disband team after review is complete

Phase 4: Invoke prose-stylist as a subagent, incorporate review results and finalize revisions
```

---

## Example 3: Webtoon Production Team (Subagent Mode)

### Team Architecture: Producer-Reviewer
### Execution Mode: Subagent

> In a Producer-Reviewer pattern with only 2 agents, where result handoff is more important than communication, subagents are the right fit.

```
Phase 1: Agent(webtoon-artist) → generate panels
Phase 2: Agent(webtoon-reviewer) → quality review
Phase 3: Agent(webtoon-artist) → regenerate problematic panels (up to 2 iterations)
```

### Agent Configuration

| Agent | subagent_type | Role | Skills |
|-------|---------------|------|--------|
| webtoon-artist | custom | Panel image generation | generate-webtoon |
| webtoon-reviewer | custom | Quality review | review-webtoon, fix-webtoon-panel |

### Full Agent File Example: `webtoon-reviewer.md`

```markdown
---
name: webtoon-reviewer
description: "Expert in reviewing the quality of webtoon panels. Evaluates composition, character consistency, text readability, and direction."
---

# Webtoon Reviewer — Webtoon Quality Review Expert

You are an expert in reviewing the quality of webtoon panels. You evaluate panels based on visual completeness, story conveyance, and character consistency.

## Core Roles
1. Evaluate the composition and visual completeness of each panel
2. Verify consistency of character appearances across panels
3. Evaluate readability and placement of speech bubble text
4. Review the direction flow and pacing of the entire episode

## Working Principles
- Render clear verdicts using the 3-tier PASS/FIX/REDO system
- FIX is for cases resolvable with partial corrections; REDO is for cases requiring full regeneration
- Judge based on objective criteria (consistency, readability, composition), not subjective preference

## Input/Output Protocol
- Input: Panel images in the `_workspace/panels/` directory
- Output: `_workspace/review_report.md`
- Format:
  ```
  ## Panel {N}
  - Verdict: PASS | FIX | REDO
  - Reason: [specific reason]
  - Revision instructions: [specific revision direction for FIX/REDO cases]
  ```

## Error Handling
- If image fails to load, mark the panel as REDO
- Panels still marked REDO after 2 regeneration attempts are force-passed with a warning

## Collaboration
- Deliver revision instructions to webtoon-artist (based on result file)
- Re-review regenerated panels (up to 2-iteration loop)
```

### Error Handling

```
Retry policy:
- Panels with REDO verdict → request regeneration from artist (with specific revision instructions)
- Force PASS after maximum 2 loops
- If 50% or more of all panels are REDO, suggest prompt revision to the user
```

---

## Example 4: Code Review Team (Agent Team Mode)

### Team Architecture: Fan-out/Fan-in + Discussion
### Execution Mode: Agent Team

> Code review is a prime example where agent teams shine. Reviewers with different perspectives share and challenge each other's findings, enabling deeper reviews.

```
[Leader] → TeamCreate(review-team)
    ├── security-reviewer: check security vulnerabilities
    ├── performance-reviewer: analyze performance impact
    └── test-reviewer: verify test coverage
    → Reviewers share findings with each other (SendMessage)
    → Leader synthesizes results
```

### Team Communication Patterns

```
security ──SendMessage──→ performance  ("This SQL query is injectable, also needs checking from a performance angle")
performance ──SendMessage──→ test      ("Found N+1 query, please check if there are related tests")
test ────SendMessage──→ security      ("No tests for auth module, what's the priority from a security perspective?")
```

Key point: Reviewers communicate **directly without going through the leader** to quickly catch cross-domain issues.

---

## Example 5: Supervisor Pattern — Code Migration Team (Agent Team Mode)

### Team Architecture: Supervisor
### Execution Mode: Agent Team

```
[supervisor/leader] → analyze file list → assign batches
    ├→ [migrator-1] (batch A)
    ├→ [migrator-2] (batch B)
    └→ [migrator-3] (batch C)
    ← receive TaskUpdate → assign additional batches or reassign
```

### Agent Configuration

| Member | Role |
|--------|------|
| (Leader = migration-supervisor) | File analysis, batch distribution, progress management |
| migrator-1~3 | Migrate assigned file batches |

### Supervisor Dynamic Distribution Logic (using Agent Team)

```
1. Collect full list of target files
2. Estimate complexity (file size, import count, dependencies)
3. Register file batches as tasks via TaskCreate (including dependencies)
4. Team members self-claim tasks
5. When a member reports completion via TaskUpdate:
   - Success → automatically claims next task
   - Failure → leader confirms cause via SendMessage → reassign or delegate to another member
6. All tasks complete → leader runs integration tests
```

Difference from Fan-out: Tasks are not fixed in advance but **dynamically assigned at runtime**. The self-claim feature of the shared task list aligns naturally with the Supervisor pattern.

---

## Output Pattern Summary

### Agent Definition Files
Location: `project/.claude/agents/{agent-name}.md`
Required sections: Core Roles, Working Principles, Input/Output Protocol, Error Handling, Collaboration
Additional section for team mode: **Team Communication Protocol** (message send/receive, task claim scope)

### Skill File Structure
Location: `project/.claude/skills/{skill-name}/SKILL.md` (project level)
Or: `~/.claude/skills/{skill-name}/SKILL.md` (global level)

### Integration Skill (Orchestrator)
A top-level skill that coordinates the entire team. Defines agent configuration and workflow for each scenario.
Template: see `references/orchestrator-template.md`.
**Execution mode must be explicitly specified** — Agent Team (default) or Subagent.
