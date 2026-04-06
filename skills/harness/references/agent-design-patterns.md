# Agent Team Design Patterns

## Execution Modes: Agent Teams vs Subagents

Understand the core differences between the two execution modes and select the appropriate one.

### Agent Teams — Default Mode

The team leader creates a team with `TeamCreate`, and team members run as independent Claude Code instances. Members communicate directly via `SendMessage` and self-coordinate using a shared task list (`TaskCreate`/`TaskUpdate`).

```
[Leader] ←→ [Member A] ←→ [Member B]
  ↕              ↕              ↕
  └──── Shared Task List ────┘
```

**Core tools:**
- `TeamCreate`: Create a team + spawn members
- `SendMessage({to: name})`: Message a specific member
- `SendMessage({to: "all"})`: Broadcast (high cost, use sparingly)
- `TaskCreate`/`TaskUpdate`: Manage the shared task list

**Characteristics:**
- Members can talk directly, challenge, and validate each other
- Members exchange information without routing through the leader
- Self-coordinate via shared task list (members can claim tasks themselves)
- Leader is automatically notified when a member becomes idle
- Plan approval mode allows review before risky operations

**Constraints:**
- Only one team can be **active** per session (teams can be dissolved and reformed between Phases)
- No nested teams (members cannot create their own teams)
- Leader is fixed (cannot be transferred)
- High token cost

**Team Reformation Pattern:**
When different expert combinations are needed per Phase, save the previous team's artifacts to files → clean up the team → create a new team. Previous team artifacts are preserved in `_workspace/`, so the new team can access them via Read.

### Subagents — Lightweight Mode

The main agent creates subagents using the `Agent` tool. Subagents return results only to the main agent and do not communicate with each other.

```
[Main] → [Sub A] → return result
       → [Sub B] → return result
       → [Sub C] → return result
```

**Core tool:**
- `Agent(prompt, subagent_type, run_in_background)`: Create a subagent

**Characteristics:**
- Lightweight and fast
- Results are returned as a summary to the main context
- Token-efficient

**Constraints:**
- No communication between subagents
- Main agent handles all coordination
- No real-time collaboration or challenge

### Mode Selection Decision Tree

```
Are there 2 or more agents?
├── Yes → Do agents need to communicate with each other?
│         ├── Yes → Agent Teams (default)
│         │         Quality improves through cross-validation, shared discoveries, and real-time feedback.
│         │
│         └── No → Subagents are also viable
│                  For produce-and-verify or expert pool patterns where only result passing is needed.
│
└── No (1 agent) → Subagent
                   A single agent does not need a team.
```

> **Core principle:** Agent Teams are the default. When choosing Subagents, ask yourself: "Is communication between members truly unnecessary?"

---

## Agent Team Architecture Types

### 1. Pipeline
Sequential task flow. The output of the previous agent becomes the input of the next.

```
[Analyze] → [Design] → [Implement] → [Verify]
```

**When appropriate:** Each stage has a strong dependency on the output of the previous stage
**Example:** Novel writing — worldbuilding → characters → plot → writing → editing
**Note:** Bottlenecks delay the entire pipeline. Design each stage to be as independent as possible.
**Team mode suitability:** Strong sequential dependency limits the benefits of team mode. However, team mode is useful if there are parallel segments within the pipeline.

### 2. Fan-out/Fan-in
Parallel processing followed by result integration. Independent tasks are performed simultaneously.

```
           ┌→ [Expert A] ─┐
[Dispatch] → ├→ [Expert B] ─┼→ [Integrate]
           └→ [Expert C] ─┘
```

**When appropriate:** Different perspectives or domain analyses are needed for the same input
**Example:** Comprehensive research — official/media/community/background simultaneous investigation → integrated report
**Note:** The quality of the integration stage determines overall quality.
**Team mode suitability:** The most natural pattern for Agent Teams. **Must be implemented as Agent Teams.** Members share discoveries and challenge each other, and one agent's finding can redirect another agent's investigation in real time, significantly improving quality compared to solo investigation.

### 3. Expert Pool
Select and call the appropriate expert based on the situation.

```
[Router] → { Expert A | Expert B | Expert C }
```

**When appropriate:** Different processing is required depending on the input type
**Example:** Code review — call only the relevant expert among security/performance/architecture specialists
**Note:** The router's classification accuracy is critical.
**Team mode suitability:** Subagents are more suitable. Since only the needed expert is called, a standing team is unnecessary.

### 4. Producer-Reviewer
A producer agent and a reviewer agent operate in pairs.

```
[Produce] → [Review] → (if issues) → [Produce] re-run
```

**When appropriate:** Output quality assurance is important and objective review criteria exist
**Example:** Webtoon — artist produces → reviewer inspects → problematic panels re-generated
**Note:** Must set a maximum retry count (2–3 times) to prevent infinite loops.
**Team mode suitability:** Agent Teams are useful. Real-time feedback is exchanged between producer and reviewer via SendMessage.

### 5. Supervisor
A central agent manages task state and dynamically distributes work to subordinate agents.

```
             ┌→ [Worker A]
[Supervisor] ─┼→ [Worker B]    ← Supervisor dynamically assigns based on state
             └→ [Worker C]
```

**When appropriate:** Workload is variable or task distribution must be decided at runtime
**Example:** Large-scale code migration — supervisor analyzes file list and assigns batches to workers
**Difference from Fan-out:** Fan-out distributes tasks ahead of time with fixed assignments; Supervisor adjusts dynamically based on progress
**Note:** Set delegation units large enough to prevent the supervisor from becoming a bottleneck.
**Team mode suitability:** The Agent Team's shared task list naturally matches the Supervisor pattern. Register tasks with TaskCreate; team members claim them themselves.

### 6. Hierarchical Delegation
Higher-level agents delegate recursively to lower-level agents. Decomposes complex problems step by step.

```
[Director] → [Team Lead A] → [Worker A1]
                           → [Worker A2]
           → [Team Lead B] → [Worker B1]
```

**When appropriate:** The problem decomposes naturally into a hierarchical structure
**Example:** Full-stack app development — Director → Frontend Lead → (UI/Logic/Tests) + Backend Lead → (API/DB/Tests)
**Note:** Depth beyond 3 levels incurs significant delays and context loss. Recommend staying within 2 levels.
**Team mode suitability:** Agent Teams cannot be nested (members cannot create their own teams). Implement the first level as a team and the second level as subagents, or flatten into a single team.

## Composite Patterns

In practice, composite patterns are more common than single patterns:

| Composite Pattern | Composition | Example |
|------------------|-------------|---------|
| **Fan-out + Producer-Reviewer** | Parallel production followed by individual review | Multilingual translation — 4 languages translated in parallel → each reviewed by native reviewers |
| **Pipeline + Fan-out** | Some stages of a sequential pipeline are parallelized | Analysis (sequential) → Implementation (parallel) → Integration test (sequential) |
| **Supervisor + Expert Pool** | Supervisor dynamically calls experts | Customer inquiry handling — Supervisor classifies inquiry and assigns to the appropriate expert |

### Execution Modes in Composite Patterns

**By default, use Agent Teams for all composite patterns.** Active communication between members is the key driver of output quality.

| Scenario | Recommended Mode | Reason |
|----------|-----------------|--------|
| **Research + Analysis** | Agent Teams | Researchers share discoveries, conflicting information discussed in real time |
| **Design + Implementation + Verification** | Agent Teams | Feedback loop between designer, implementer, and verifier |
| **Supervisor + Workers** | Agent Teams | Dynamic assignment via shared task list, workers share progress |
| **Produce + Review** | Agent Teams | Rework minimized through real-time feedback between producer and reviewer |

> Consider mixing in Subagents only when a single agent performs a fully isolated, one-shot task.

## Agent Type Selection

When invoking an agent, specify the type using the `subagent_type` parameter of the Agent tool. Team members in Agent Teams can also use custom agent definitions.

### Built-in Types

| Type | Tool Access | Best For |
|------|-------------|----------|
| `general-purpose` | Full (including WebSearch, WebFetch) | Web research, general-purpose tasks |
| `Explore` | Read-only (no Edit/Write) | Codebase exploration, analysis |
| `Plan` | Read-only (no Edit/Write) | Architecture design, planning |

### Custom Types

Define an agent in `.claude/agents/{name}.md` and invoke it with `subagent_type: "{name}"`. Custom agents have access to all tools.

### Selection Criteria

| Situation | Recommended | Reason |
|-----------|-------------|--------|
| Complex role reused across multiple sessions | **Custom type** (`.claude/agents/`) | Persona and task principles managed as a file |
| Simple investigation/collection, prompt alone is sufficient | **`general-purpose`** + detailed prompt | No agent file needed; include instructions in the prompt |
| Only needs to read code (analysis/review) | **`Explore`** | Prevents accidental file modification |
| Only needs design/planning | **`Plan`** | Focused on analysis; prevents code changes |
| Implementation task requiring file modification | **Custom type** | Full tool access + specialized instructions |

**Principle:** Every agent must be defined as a `.claude/agents/{name}.md` file. Even for built-in types, create an agent definition file to specify the role, principles, and protocols. The file must exist for reuse in future sessions, and the team communication protocol must be specified to ensure collaboration quality.

**Model:** All agents use `model: "opus"`. Always specify the `model: "opus"` parameter when calling the Agent tool.

## Agent Definition Structure

```markdown
---
name: agent-name
description: "1-2 sentence role description. List trigger keywords."
---

# Agent Name — One-line Role Summary

You are an expert [role] in [domain].

## Core Responsibilities
1. Responsibility 1
2. Responsibility 2

## Task Principles
- Principle 1
- Principle 2

## Input/Output Protocol
- Input: [what is received and from where]
- Output: [what is written and where]
- Format: [file format, structure]

## Team Communication Protocol (Agent Team Mode)
- Receive messages: [from whom and what type of messages]
- Send messages: [to whom and what type of messages]
- Task requests: [what types of tasks are claimed from the shared task list]

## Error Handling
- [Action on failure]
- [Action on timeout]

## Collaboration
- Relationships with other agents
```

## Agent Separation Criteria

| Criterion | Separate | Merge |
|-----------|----------|-------|
| Expertise | Separate if domains differ | Merge if domains overlap |
| Parallelism | Separate if independently executable | Consider merging if sequentially dependent |
| Context | Separate if context burden is high | Merge if lightweight and fast |
| Reusability | Separate if used by other teams | Consider merging if only used by this team |

## Skills vs Agents

| | Skill | Agent |
|-|-------|-------|
| Definition | Procedural knowledge + tool bundle | Expert persona + behavioral principles |
| Location | `.claude/skills/` | `.claude/agents/` |
| Trigger | User request keyword matching | Explicit invocation via Agent tool |
| Size | Small to large (workflows) | Small (role definitions) |
| Purpose | "How to do it" | "Who does it" |

Skills are **procedural guides** that agents reference when performing tasks.
Agents are **expert role definitions** that utilize skills.

## Skill ↔ Agent Connection Methods

Three ways an agent can utilize a skill:

| Method | Implementation | When Appropriate |
|--------|---------------|-----------------|
| **Skill tool invocation** | Specify `invoke /skill-name via the Skill tool` in the agent prompt | When the skill is an independent workflow and can be user-invoked |
| **Inline in prompt** | Include the skill content directly in the agent definition | When the skill is short (50 lines or fewer) and exclusive to this agent |
| **Reference load** | Load the skill's references/ file on demand via `Read` | When the skill content is large and only conditionally needed |

Recommendation: Use the Skill tool for high reusability, inline for dedicated use, and reference load for large content.
