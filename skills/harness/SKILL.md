---
name: harness
description: "Configures a harness. A meta-skill that defines specialized agents and creates the skills those agents will use. Trigger when: (1) user requests 'set up a harness' or 'build a harness', (2) user requests 'harness design' or 'harness engineering', (3) building a harness-based automation system for a new domain or project, (4) reconfiguring or extending an existing harness, (5) requests for 'harness inspection', 'harness audit', 'harness status', or 'agent/skill sync' for operations and maintenance of an existing harness."
---

# Harness — Agent Team & Skill Architect

A meta-skill for configuring a harness tailored to a domain or project, defining each agent's role, and creating the skills those agents will use.

**Core Principles:**
1. Create agent definitions (`.claude/agents/`) and skills (`.claude/skills/`).
2. **Use Agent Teams as the default execution mode.**
3. **Register the harness context in CLAUDE.md.** — Record the harness structure and trigger rules in the project's CLAUDE.md so that agent teams activate immediately even in new sessions.
4. **The harness is an evolving system, not a fixed artifact.** — Incorporate feedback after each execution and continuously update agents, skills, and CLAUDE.md.

## Workflow

### Phase 0: Status Audit

When the harness skill is triggered, first check the current state of any existing harness.

1. Read `project/.claude/agents/`, `project/.claude/skills/`, and `project/CLAUDE.md`
2. Branch execution mode based on current state:
   - **New build**: Agent/skill directories do not exist or are empty → run all phases starting from Phase 1
   - **Existing extension**: An existing harness is present and the request is to add new agents/skills → run only the necessary phases per the Phase Selection Matrix below
   - **Operations/maintenance**: Request to audit, modify, or sync an existing harness → proceed to the Phase 7-5 Operations/Maintenance Workflow

   **Phase Selection Matrix for Existing Extensions:**
   | Change Type | Phase 1 | Phase 2 | Phase 3 | Phase 4 | Phase 5 | Phase 6 |
   |-------------|---------|---------|---------|---------|---------|---------|
   | Add agent | Skip (use Phase 0 results) | Placement decision only | Required | If dedicated skill needed | Modify orchestrator | Required |
   | Add/modify skill | Skip | Skip | Skip | Required | If connections change | Required |
   | Architecture change | Skip | Required | Affected agents only | Affected skills only | Required | Required |
3. Compare the existing agent/skill list against CLAUDE.md records to detect drift
4. Report the audit results to the user and confirm the execution plan

### Phase 1: Domain Analysis
1. Identify the domain/project from the user's request
2. Identify core task types (creation, validation, editing, analysis, etc.)
3. Analyze conflicts/overlaps with existing agents/skills based on the Phase 0 audit
4. Explore the project codebase — identify the tech stack, data models, and key modules
5. **Detect user proficiency** — infer technical level from contextual cues in the conversation (terminology used, question depth) and adjust communication tone accordingly. Do not use terms like "assertion" or "JSON schema" without explanation when speaking with users who have limited coding experience.

### Phase 2: Team Architecture Design

#### 2-1. Execution Mode Selection: Agent Team vs Sub-Agent

**The default is Agent Team.** When two or more agents collaborate, prefer Agent Teams. Team members coordinate autonomously via direct communication (SendMessage) and a shared task list (TaskCreate); sharing findings, discussing trade-offs, and filling gaps in each other's work improves output quality.

Use Sub-Agent mode only when there is a single agent or when inter-agent communication is unnecessary (only result delivery is needed).

> See the "Execution Modes" section in `references/agent-design-patterns.md` for the comparison table and decision tree.

#### 2-2. Architecture Pattern Selection

1. Decompose the task into specialized domains
2. Determine the agent team structure (see `references/agent-design-patterns.md` for architecture patterns)
   - **Pipeline**: Sequentially dependent tasks
   - **Fan-out/Fan-in**: Parallel independent tasks
   - **Expert Pool**: Contextual selective invocation
   - **Producer-Reviewer**: Generation followed by quality review
   - **Supervisor**: A central agent manages state and dynamically distributes work
   - **Hierarchical Delegation**: Higher-level agents recursively delegate to lower-level agents

#### 2-3. Agent Separation Criteria

Evaluate across four axes: specialization, parallelism, context, and reusability. See the "Agent Separation Criteria" section in `references/agent-design-patterns.md` for the detailed criteria table.

### Phase 3: Agent Definition Creation

**Every agent must be defined as a file at `project/.claude/agents/{name}.md`.** Directly embedding role descriptions in the Agent tool's prompt without a definition file is prohibited. Reasons:
- Agent definitions must exist as files to be reusable in future sessions
- Team communication protocols must be explicit to guarantee inter-agent collaboration quality
- The core value of a harness is the separation of agents (who) from skills (how)

Even when using built-in types (`general-purpose`, `Explore`, `Plan`), create an agent definition file. Specify the built-in type via the `subagent_type` parameter of the Agent tool, and put the role, principles, and protocols in the definition file.

**Model setting:** All agents use `model: "opus"`. Always specify the `model: "opus"` parameter when calling the Agent tool. Harness quality is directly tied to the agent's reasoning capability, and opus guarantees the highest quality.

**Team reconstitution:** Only one team can be active per session, but you can dissolve a team between phases and form a new one. For pipeline patterns that require different specialist combinations per phase, save the previous team's outputs to files, clean up the team, and form a new team.

Define each agent in `project/.claude/agents/{name}.md`. Required sections: core role, working principles, input/output protocol, error handling, and collaboration. In agent team mode, add a `## Team Communication Protocol` section that specifies message sender/receiver relationships and the scope of task requests.

> See the "Agent Definition Structure" in `references/agent-design-patterns.md` and `references/team-examples.md` for definition templates and complete file examples.

**Interim CLAUDE.md Sync (on Phase 3 completion):**
Immediately after Phase 3, update the agent list table in CLAUDE.md. Reflect newly added agents immediately; also sync deletions and changes. This is an **interim sync** to guard against session interruption — the full context is finalized in Phase 5-4.

**Requirements when including a QA agent:**
- Use the `general-purpose` type for QA agents (`Explore` is read-only and cannot run validation scripts)
- The essence of QA is not "existence checking" but **"cross-boundary comparison"** — read API responses and frontend hooks simultaneously and compare shapes
- Run QA not once at the end, but **incrementally, immediately after each module is complete** (incremental QA)
- Detailed guide: see `references/qa-agent-guide.md`

### Phase 4: Skill Creation

Create the skills each agent will use at `project/.claude/skills/{name}/SKILL.md`. See `references/skill-writing-guide.md` for detailed authoring guidelines.

#### 4-1. Skill Structure

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description required)
│   └── Markdown body
└── Bundled Resources (optional)
    ├── scripts/    - executable code for repetitive/deterministic tasks
    ├── references/ - reference documents loaded conditionally
    └── assets/     - files used in output (templates, images, etc.)
```

#### 4-2. Writing the Description — Drive Active Triggering

The description is the only trigger mechanism for a skill. Because Claude tends to trigger conservatively, write descriptions in an **assertive ("pushy")** manner.

**Bad example:** `"A skill for processing PDF documents"`
**Good example:** `"Handles all PDF tasks: reading PDF files, extracting text/tables, merging, splitting, rotating, watermarking, encrypting, OCR, and more. Whenever a .pdf file is mentioned or a PDF output is requested, this skill MUST be used."`

Key: describe both what the skill does and the specific triggering situations, and differentiate it from similar cases that should NOT trigger this skill.

#### 4-3. Body Writing Principles

| Principle | Description |
|-----------|-------------|
| **Explain the Why** | Instead of coercive directives like "ALWAYS/NEVER", convey the reason why something should be done. When an LLM understands the reasoning, it makes correct judgments even in edge cases. |
| **Stay Lean** | The context window is a shared resource. Target under 500 lines for the SKILL.md body; remove or move to references/ anything that doesn't pull its weight. |
| **Generalize** | Explain the underlying principle rather than narrow rules that only fit specific examples, so the skill handles a wide variety of inputs. No overfitting. |
| **Bundle Repetitive Code** | If agents are writing the same scripts repeatedly during test runs, pre-bundle that code in `scripts/`. |
| **Write in Imperative Voice** | Use direct, instructional language ("Check…", "Create…", "Verify…"). |

#### 4-4. Progressive Disclosure

Skills manage context through a three-tier loading system:

| Tier | When Loaded | Size Target |
|------|-------------|-------------|
| **Metadata** (name + description) | Always in context | ~100 words |
| **SKILL.md body** | When the skill is triggered | <500 lines |
| **references/** | Only when needed | Unlimited (scripts can be executed without loading) |

**Size management rules:**
- When SKILL.md approaches 500 lines, move details to references/ and leave a pointer in the body explaining "when to read this file"
- Include a **Table of Contents (ToC)** at the top of any reference file over 300 lines
- If there are domain- or framework-specific variants, separate them into per-domain files under references/ so only the relevant file is loaded

```
cloud-deploy/
├── SKILL.md (workflow + selection guide)
└── references/
    ├── aws.md    ← load only when AWS is selected
    ├── gcp.md
    └── azure.md
```

#### 4-5. Skill-Agent Connection Principles

- 1 agent ↔ 1–N skills (1:1 or 1:many)
- A skill can be shared by multiple agents
- Skills carry "how to do it"; agents carry "who does it"

> See `references/skill-writing-guide.md` for detailed writing patterns, examples, and data schema standards.

**Interim CLAUDE.md Sync (on Phase 4 completion):**
Immediately after Phase 4, update the skill list and directory structure in CLAUDE.md. Reflect newly created skill directories in the directory tree immediately. Like Phase 3, this is an **interim sync** to guard against session interruption; finalized in Phase 5-4.

### Phase 5: Integration and Orchestration

The orchestrator is a specialized form of skill that ties individual agents and skills into a single workflow and coordinates the entire team. If Phase 4 skills define "what each agent does and how", the orchestrator defines "who collaborates, when, and in what order". See `references/orchestrator-template.md` for concrete templates.

**Modifying the orchestrator for existing extensions:** When extending rather than building from scratch, modify the existing orchestrator rather than creating a new one. When adding agents, reflect them in team composition, task assignment, and data flow, and add trigger keywords related to the new agent to the description.

The orchestrator pattern varies by execution mode:

#### 5-0. Orchestrator Patterns by Mode

**Agent Team Mode (default):**
The orchestrator forms a team with `TeamCreate` and assigns tasks with `TaskCreate`. Team members communicate directly via `SendMessage` and coordinate autonomously. The leader (orchestrator) monitors progress and consolidates results.

```
[Orchestrator/Leader]
    ├── TeamCreate(team_name, members)
    ├── TaskCreate(tasks with dependencies)
    ├── Team members coordinate autonomously (SendMessage)
    ├── Collect and consolidate results
    └── Clean up team
```

**Sub-Agent Mode:**
The orchestrator calls sub-agents directly using the `Agent` tool. Sub-agents return results only to the main orchestrator.

```
[Orchestrator]
    ├── Agent(agent-1, run_in_background=true)
    ├── Agent(agent-2, run_in_background=true)
    ├── Wait for and collect results
    └── Generate integrated output
```

#### 5-1. Data Transfer Protocol

Specify how data is passed between agents within the orchestrator:

| Strategy | Method | Execution Mode | Best For |
|----------|--------|---------------|----------|
| **Message-based** | Direct communication between team members via `SendMessage` | Agent Team | Real-time coordination, feedback exchange, lightweight state passing |
| **Task-based** | Share task state via `TaskCreate`/`TaskUpdate` | Agent Team | Progress tracking, dependency management, task-level requests |
| **File-based** | Write and read files at agreed-upon paths | Both | Large data, structured artifacts, audit trail requirements |

**Recommended combination for Agent Team mode:** Task-based (coordination) + File-based (artifacts) + Message-based (real-time communication)

File-based transfer rules:
- Create a `_workspace/` folder under the working directory to store intermediate artifacts
- File naming convention: `{phase}_{agent}_{artifact}.{ext}` (e.g., `01_analyst_requirements.md`)
- Output only final artifacts to the user-specified path; preserve intermediate files (`_workspace/`) for post-hoc verification and audit tracing

#### 5-2. Error Handling

Include an error handling policy within the orchestrator. Core principle: retry once, and if it fails again, proceed without that result (note the omission in the report); do not discard conflicting data — include source attribution.

> See the "Error Handling" section of `references/orchestrator-template.md` for per-error-type strategy tables and implementation details.

#### 5-3. Team Mode Only: Team Size Guidelines

| Task Scale | Recommended Team Size | Tasks per Member |
|------------|----------------------|-----------------|
| Small (5–10 tasks) | 2–3 members | 3–5 tasks |
| Medium (10–20 tasks) | 3–5 members | 4–6 tasks |
| Large (20+ tasks) | 5–7 members | 4–5 tasks |

> More team members means more coordination overhead. Three focused members outperform five distracted ones.

#### 5-4. Registering Harness Context in CLAUDE.md

After completing the harness configuration, register the harness context in the project's `CLAUDE.md`. Because CLAUDE.md is always loaded when a new session starts, the harness's existence and usage rules must be recorded there so agent teams function correctly in subsequent sessions.

**What to record in CLAUDE.md (no duplication with the orchestrator):**

````markdown
## Harness: {domain name}

**Goal:** {one-line core goal of the harness}

**Agent Team:**
| Agent | Role |
|-------|------|
| {name} | {one-line role description} |

**Skills:**
| Skill | Purpose | Using Agent |
|-------|---------|-------------|
| {skill-name} | {one-line description} | {agent-name} |

**Execution Rules:**
- For {domain}-related task requests, process via the `{orchestrator-skill-name}` skill using the agent team
- Simple questions/confirmations can be answered directly without engaging the agent team
- All agents use `model: "opus"`
- Intermediate artifacts: `_workspace/` directory

**Directory Structure:**
```
.claude/
├── agents/
│   └── {agent-name}.md
└── skills/
    └── {skill-name}/
        ├── SKILL.md
        └── references/
```

**Change History:**
| Date | Change | Target | Reason |
|------|--------|--------|--------|
| {YYYY-MM-DD} | Initial setup | All | - |
````

**Role Division Between CLAUDE.md and the Orchestrator:**

| Item | CLAUDE.md | Orchestrator Skill |
|------|----------|--------------------|
| Harness existence notification | Yes | No |
| Agent list (name + one-line role) | Yes | Yes (detailed) |
| Skill list (name + purpose + agent) | Yes | Yes (detailed) |
| Directory structure | Yes | No |
| Trigger rules | Yes (when to use the skill) | No (handled by description) |
| Workflow details | No | Yes |
| Data flow | No | Yes |
| Error handling | No | Yes |
| Change history | Yes | No |

Key: CLAUDE.md only carries "the harness exists, use it when…"; delegate "how to execute" to the orchestrator.

#### 5-5. Follow-up Task Support

The orchestrator must handle follow-up tasks in addition to the initial run. Ensure the following three things:

**1. Include follow-up keywords in the orchestrator description:**
Initial creation keywords alone will not trigger follow-up requests. Always include these follow-up expressions in the description:
- "re-run", "run again", "update", "revise", "supplement"
- "redo only the {sub-task} of {domain}"
- "based on previous results", "improve results"

**2. Add a context-check step to orchestrator Phase 1:**
At the start of the workflow, check whether previous artifacts exist to determine the execution mode:
- `_workspace/` exists + user requests partial modification → **partial re-run** (re-invoke only the relevant agent)
- `_workspace/` exists + user provides new input → **new run** (move existing `_workspace/` to `_workspace_prev/`)
- `_workspace/` does not exist → **initial run**

**3. Include re-invocation guidance in agent definitions:**
Specify in each agent's `.md` file "how to behave when previous artifacts exist":
- If a previous result file exists, read it and incorporate improvements
- If user feedback is provided, modify only the relevant part

> See the "Phase 0: Context Check" section of the orchestrator template: `references/orchestrator-template.md`

### Phase 6: Validation and Testing

Validate the generated harness. See `references/skill-testing-guide.md` for detailed testing methodology.

#### 6-1. Structure Validation

- Verify all agent files are in the correct locations
- Validate skill frontmatter (name, description)
- Check inter-agent reference consistency
- Confirm no commands were generated

#### 6-2. Execution Mode Validation

- Agent Team mode: check communication paths between team members, task dependencies, and team size appropriateness
- Sub-Agent mode: verify input/output connections for each agent and `run_in_background` settings

#### 6-3. Skill Execution Testing

Perform actual execution tests for each generated skill:

1. **Write test prompts** — Write 2–3 realistic test prompts for each skill. Use concrete, natural sentences that actual users would likely type.

2. **With-skill vs Without-skill comparison** — Where possible, run with-skill and without-skill executions in parallel to verify the skill's added value. Spawn two sub-agents for each:
   - **With-skill**: Read the skill and perform the task
   - **Without-skill (baseline)**: Perform the same prompt without the skill

3. **Evaluate results** — Assess output quality both qualitatively (user review) and quantitatively (assertion-based). Define assertions when output is objectively verifiable (file creation, data extraction, etc.); rely on user feedback for subjective cases (tone, design).

4. **Iterative improvement loop** — When issues are found in test results:
   - **Generalize** the feedback before modifying the skill (no narrow fixes that only address specific examples)
   - Re-test after modification
   - Repeat until the user is satisfied or no further meaningful improvement is possible

5. **Bundle repetitive patterns** — If agents are writing the same code across test runs (e.g., the same helper script in every test), pre-bundle that code in `scripts/`.

#### 6-4. Trigger Validation

Validate that each skill's description triggers correctly:

1. **Should-trigger queries** (8–10) — A variety of expressions that should trigger the skill (formal/casual, explicit/implicit)
2. **Should-NOT-trigger queries** (8–10) — "Near-miss" queries where keywords are similar but a different tool/skill is the appropriate choice

**Key for writing near-misses:** Queries obviously unrelated like "write a Fibonacci function" have no test value. Good test cases are **boundary-ambiguous queries** such as "extract the chart from this Excel file as a PNG" (xlsx skill vs image conversion).

Also check for trigger conflicts with existing skills at this stage.

#### 6-5. Dry-Run Testing

- Review whether the orchestrator skill's phase order is logically sound
- Check for dead links in data transfer paths
- Confirm each agent's inputs match the outputs of the previous phase
- Verify that fallback paths for each error scenario are executable

#### 6-6. Test Scenario Writing

- Add a `## Test Scenarios` section to the orchestrator skill
- Describe at least one normal flow and one error flow

### Phase 7: Harness Evolution

The harness is not a static artifact built once and left alone. It is a system that continuously evolves based on user feedback.

#### 7-1. Collect Post-Run Feedback

After each harness execution, request feedback from the user:
- "Are there any areas in the results that could be improved?"
- "Are there any changes you'd like to make to the agent team structure or workflow?"

If there is no feedback, move on. Do not pressure users, but always offer the opportunity.

#### 7-2. Feedback Incorporation Paths

The target for modification depends on the type of feedback:

| Feedback Type | Modification Target | Example |
|---------------|---------------------|---------|
| Output quality | The relevant agent's skill | "The analysis is too superficial" → add depth criteria to the skill |
| Agent role | Agent definition `.md` | "We also need a security review" → add a new agent |
| Workflow order | Orchestrator skill | "Validation should come first" → reorder phases |
| Team composition | Orchestrator + agents | "These two could be merged" → merge agents |
| Missing trigger | Skill description | "This phrasing doesn't work" → expand the description |

#### 7-3. Change History

Record all changes in the **Change History** table in CLAUDE.md (same table as the "Change History" section in the Phase 5-4 template):

```markdown
**Change History:**
| Date | Change | Target | Reason |
|------|--------|--------|--------|
| 2026-04-05 | Initial setup | All | - |
| 2026-04-07 | Added QA agent | agents/qa.md | Feedback: insufficient output quality validation |
| 2026-04-10 | Added tone guide | skills/content-creator | Feedback: "too stiff" |
```

Use this history to track the direction in which the harness has evolved and to prevent regressions.

#### 7-4. Evolution Triggers

Propose evolution not only when the user explicitly says "update the harness", but also in these situations:
- The same type of feedback has been repeated two or more times
- A pattern of repeated agent failures is detected
- The user is observed manually working around the orchestrator

#### 7-5. Operations/Maintenance Workflow

Systematically perform inspection, modification, and synchronization of an existing harness. Follow this workflow when the Phase 0 audit routes to the "operations/maintenance" branch.

**Step 1: Status Audit**
- Compare `.claude/agents/` file list vs CLAUDE.md agent table → generate discrepancy list
- Compare `.claude/skills/` directory list vs CLAUDE.md skill table → generate discrepancy list
- Compare CLAUDE.md directory structure vs actual file system → detect drift
- Report audit results to the user

**Step 2: Incremental Additions/Modifications**
- Per user request, add/modify/delete agents and add/modify/delete skills
- Make one change at a time; immediately run Step 3 (sync) after each change

**Step 3: CLAUDE.md Synchronization**
- Update the agent table, skill table, directory structure, and change history to reflect the actual state
- Record date, change description, target, and reason in the change history

**Step 4: Change Validation**
- Validate the structure of modified agents/skills (per Phase 6-1 criteria)
- If the change scope affects triggers, run trigger validation (per Phase 6-4 criteria)
- For large-scale changes (architecture changes, adding/removing 3+ agents), also run Phase 6-3 (execution testing) and Phase 6-5 (dry-run)
- Final verification that CLAUDE.md and actual files are in sync

## Output Checklist

Verify after completion:

- [ ] `project/.claude/agents/` — **Agent definition files must be created** (required even for built-in types)
- [ ] `project/.claude/skills/` — Skill files (SKILL.md + references/)
- [ ] One orchestrator skill (including data flow + error handling + test scenarios)
- [ ] Execution mode specified (Agent Team or Sub-Agent)
- [ ] All Agent calls include the `model: "opus"` parameter
- [ ] `.claude/commands/` — Nothing created here
- [ ] No conflicts with existing agents/skills
- [ ] Skill descriptions are assertive ("pushy") — **including follow-up keywords**
- [ ] SKILL.md body is under 500 lines; move to references/ if exceeded
- [ ] Execution verified with 2–3 test prompts
- [ ] Trigger validation complete (should-trigger + should-NOT-trigger)
- [ ] **Harness context registered in CLAUDE.md** (agent list, skill list, execution rules, change history)
- [ ] **Agents/skills/directory structure/key reference paths reflected in CLAUDE.md** — sync immediately upon Phase 3 and 4 completion
- [ ] **Agent/skill additions/deletions/modifications recorded in CLAUDE.md change history**
- [ ] **Context-check step in orchestrator Phase 1** (determine initial/follow-up/partial re-run)

## References

- Harness patterns: `references/agent-design-patterns.md`
- Existing harness examples (complete file contents): `references/team-examples.md`
- Orchestrator template: `references/orchestrator-template.md`
- **Skill writing guide**: `references/skill-writing-guide.md` — writing patterns, examples, and data schema standards
- **Skill testing guide**: `references/skill-testing-guide.md` — testing, evaluation, and iterative improvement methodology
- **QA agent guide**: `references/qa-agent-guide.md` — reference when including a QA agent in a build harness. Contains integration consistency validation methodology, boundary bug patterns, and QA agent definition templates. Based on 7 real-world bug cases from actual projects.
