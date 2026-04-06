# Orchestrator Skill Template

The Orchestrator is the top-level skill that coordinates the entire team. Two templates are provided depending on the execution mode.

---

## Template A: Agent Team Mode (Default)

Agent Teams are formed with `TeamCreate` and coordinated via a shared task list and `SendMessage`.

```markdown
---
name: {domain}-orchestrator
description: "Orchestrator that coordinates the {domain} agent team. {initial trigger keywords}. Follow-up tasks: also use this skill when asked to modify {domain} results, partially re-run, update, supplement, re-execute, or improve previous results."
---

# {Domain} Orchestrator

An integrated skill that coordinates the {domain} agent team to produce {final deliverable}.

## Execution Mode: Agent Teams

## Agent Composition

| Member | Agent Type | Role | Skill | Output |
|--------|-----------|------|-------|--------|
| {teammate-1} | {custom or built-in} | {role} | {skill} | {output-file} |
| {teammate-2} | {custom or built-in} | {role} | {skill} | {output-file} |
| ... | | | | |

## Workflow

### Phase 0: Context Check (Follow-up Support)

Check whether existing artifacts exist to determine the execution mode:

1. Check whether the `_workspace/` directory exists
2. Determine execution mode:
   - **`_workspace/` does not exist** → Initial run. Proceed to Phase 1
   - **`_workspace/` exists + user requests partial modification** → Partial re-run. Re-invoke only the relevant agent and overwrite only the targeted artifacts from the existing output
   - **`_workspace/` exists + new input provided** → New run. Move existing `_workspace/` to `_workspace_{YYYYMMDD_HHMMSS}/`, then proceed to Phase 1
3. On partial re-run: include the path to previous artifacts in the agent prompt so the agent reads existing results and incorporates feedback

### Phase 1: Preparation
1. Analyze user input — {what to identify}
2. Create `_workspace/` in the working directory (on initial run)
3. Save input data to `_workspace/00_input/`

### Phase 2: Team Setup

1. Create team:
   ```
   TeamCreate(
     team_name: "{domain}-team",
     members: [
       { name: "{teammate-1}", agent_type: "{type}", model: "opus", prompt: "{role description and task instructions}" },
       { name: "{teammate-2}", agent_type: "{type}", model: "opus", prompt: "{role description and task instructions}" },
       ...
     ]
   )
   ```

2. Register tasks:
   ```
   TaskCreate(tasks: [
     { title: "{task 1}", description: "{details}", assignee: "{teammate-1}" },
     { title: "{task 2}", description: "{details}", assignee: "{teammate-2}" },
     { title: "{task 3}", description: "{details}", depends_on: ["{task 1}"] },
     ...
   ])
   ```

   > 5–6 tasks per member is appropriate. Specify dependencies with `depends_on`.

### Phase 3: {Main Work — e.g., Research/Generation/Analysis}

**Execution approach:** Members self-coordinate

Members claim tasks from the shared task list and perform them independently.
The leader monitors progress and intervenes when necessary.

**Inter-member communication rules:**
- {teammate-1} sends {what information} to {teammate-2} via SendMessage
- {teammate-2} saves results to a file upon task completion and notifies the leader
- If a member needs another member's result, they request it via SendMessage

**Artifact storage:**

| Member | Output path |
|--------|------------|
| {teammate-1} | `_workspace/{phase}_{teammate-1}_{artifact}.md` |
| {teammate-2} | `_workspace/{phase}_{teammate-2}_{artifact}.md` |

**Leader monitoring:**
- Automatically notified when a member becomes idle
- Send instructions or reassign tasks via SendMessage when a member is blocked
- Check overall progress with TaskGet

### Phase 4: {Follow-up Work — e.g., Verification/Integration}
1. Wait for all members to complete their tasks (check status with TaskGet)
2. Collect each member's artifacts via Read
3. {Integration/verification logic}
4. Generate final deliverable: `{output-path}/{filename}`

### Phase 5: Cleanup
1. Request members to terminate (SendMessage)
2. Clean up the team (TeamDelete)
3. Preserve the `_workspace/` directory (do not delete intermediate artifacts — for post-hoc verification and audit trail)
4. Report results summary to the user

> **When team reformation is needed:** If a different expert combination is needed per Phase, clean up the current team with TeamDelete, then create the next Phase's team with a new TeamCreate. Previous team artifacts are preserved in `_workspace/`, so the new team can access them via Read.

## Data Flow

```
[Leader] → TeamCreate → [teammate-1] ←SendMessage→ [teammate-2]
                            │                           │
                            ↓                           ↓
                      artifact-1.md              artifact-2.md
                            │                           │
                            └───────── Read ────────────┘
                                       ↓
                                [Leader: Integrate]
                                       ↓
                                Final Deliverable
```

## Error Handling

| Situation | Strategy |
|-----------|----------|
| 1 member fails/stops | Leader detects → checks status via SendMessage → restarts or creates a replacement member |
| Majority of members fail | Notify user and confirm whether to proceed |
| Timeout | Use partial results collected so far; terminate incomplete members |
| Data conflict between members | Include source attribution and present both; do not delete |
| Task status delayed | Leader checks with TaskGet and manually updates with TaskUpdate |

## Test Scenarios

### Happy Path
1. User provides {input}
2. Phase 1 produces {analysis result}
3. Phase 2 forms the team ({N} members + {M} tasks)
4. Phase 3 members self-coordinate and perform tasks
5. Phase 4 integrates artifacts to produce final result
6. Phase 5 cleans up the team
7. Expected result: `{output-path}/{filename}` created

### Error Path
1. {teammate-2} stops due to an error in Phase 3
2. Leader receives idle notification
3. Check status via SendMessage → attempt restart
4. If restart fails, reassign {teammate-2}'s tasks to {teammate-1}
5. Proceed to Phase 4 with remaining results
6. State "{teammate-2} area partially uncollected" in the final report
```

---

## Template B: Subagent Mode (Lightweight)

Subagents are invoked directly with the `Agent` tool and return results only to the main agent.

```markdown
---
name: {domain}-orchestrator
description: "Orchestrator that coordinates {domain} agents. {initial trigger keywords}. Follow-up tasks: also use this skill when asked to modify {domain} results, partially re-run, update, supplement, re-execute, or improve previous results."
---

# {Domain} Orchestrator

An integrated skill that coordinates {domain} agents to produce {final deliverable}.

## Execution Mode: Subagents

## Agent Composition

| Agent | subagent_type | Role | Skill | Output |
|-------|--------------|------|-------|--------|
| {agent-1} | {custom or built-in type} | {role} | {skill} | {output-file} |
| {agent-2} | {custom or built-in type} | {role} | {skill} | {output-file} |
| ... | | | | |

## Workflow

### Phase 0: Context Check (Follow-up Support)

Check whether existing artifacts exist to determine the execution mode:

1. Check whether the `_workspace/` directory exists
2. Determine execution mode:
   - **`_workspace/` does not exist** → Initial run. Proceed to Phase 1
   - **`_workspace/` exists + user requests partial modification** → Partial re-run. Re-invoke only the relevant agent
   - **`_workspace/` exists + new input provided** → New run. Move existing `_workspace/` to `_workspace_{YYYYMMDD_HHMMSS}/`

### Phase 1: Preparation
1. Analyze user input — {what to identify}
2. Create `_workspace/` in the working directory (on initial run)
3. Save input data to `_workspace/00_input/`

### Phase 2: {Main Work — e.g., Research/Generation/Analysis}

**Execution approach:** {Parallel | Sequential | Conditional}

{If parallel}
Invoke N Agent tools simultaneously in a single message:

| Agent | Input | Output | model | run_in_background |
|-------|-------|--------|-------|-------------------|
| {agent-1} | {input source} | `_workspace/{phase}_{agent}_{artifact}.md` | opus | true |
| {agent-2} | {input source} | `_workspace/{phase}_{agent}_{artifact}.md` | opus | true |

{If sequential}
Pass the output of the previous agent as input to the next:

1. Run {agent-1} → produce `_workspace/01_{artifact}.md`
2. Run {agent-2} (input: output of 01) → produce `_workspace/02_{artifact}.md`

### Phase 3: {Follow-up Work — e.g., Verification/Integration}
1. Collect Phase 2 artifacts via Read
2. {Integration/verification logic}
3. Generate final deliverable: `{output-path}/{filename}`

### Phase 4: Cleanup
1. Preserve the `_workspace/` directory (do not delete intermediate artifacts — for post-hoc verification and audit trail)
2. Report results summary to the user

## Data Flow

```
Input → [agent-1] → artifact-1 ─┐
                                 ├→ [Integrate] → Final Deliverable
Input → [agent-2] → artifact-2 ─┘
```

## Error Handling

| Situation | Strategy |
|-----------|----------|
| 1 agent fails | Retry once. If it fails again, proceed without that result and note the omission in the report |
| Majority of agents fail | Notify user and confirm whether to proceed |
| Timeout | Use partial results collected so far |
| Data conflict between agents | Include source attribution and present both; do not delete |

## Test Scenarios

### Happy Path
1. User provides {input}
2. Phase 1 produces {analysis result}
3. Phase 2 runs {N} agents in parallel, each producing artifacts
4. Phase 3 integrates artifacts to produce the final report
5. Expected result: `{output-path}/{filename}` created

### Error Path
1. {agent-2} fails in Phase 2
2. Still fails after one retry
3. Proceed to Phase 3 with remaining results, excluding {agent-2}'s result
4. State "{agent-2} area data uncollected" in the final report
5. Notify user of partial completion
```

---

## Authoring Principles

1. **Declare the execution mode first** — State "Agent Teams" or "Subagents" at the top of the Orchestrator
2. **Be specific about TeamCreate/SendMessage/TaskCreate usage in Agent Team mode** — team formation, task registration, communication rules
3. **Specify all parameters of the Agent tool in Subagent mode** — name, subagent_type, prompt, run_in_background
4. **File paths must be absolute** — no relative paths; use clear paths based on `_workspace/`
5. **Declare inter-Phase dependencies** — specify which Phase depends on the results of which other Phase
6. **Error handling must be realistic** — do not assume "everything will succeed"
7. **Test scenarios are mandatory** — at least 1 happy path + 1 error path

## Follow-up Keywords for the description Field

An Orchestrator description with only initial trigger keywords is insufficient. Always include the following follow-up expressions:

- re-run / run again / update / modify / supplement
- "just {part} of {domain} again"
- "based on previous results", "improve results"
- Domain-related everyday requests (e.g., for a launch strategy harness: "launch", "promotion", "trending", etc.)

Without follow-up keywords, the harness becomes effectively dead code after the first run.

## Reference Orchestrators

Basic structure of a Fan-out/Fan-in pattern Orchestrator:
Preparation → Phase 0 (context check) → TeamCreate + TaskCreate → N members run in parallel → Read + integrate → cleanup.
See the research team example in `references/team-examples.md`.
