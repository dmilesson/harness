# Skill Testing & Iterative Improvement Guide

A methodology for validating quality and iteratively improving skills created in the harness. Supplementary reference for SKILL.md Phase 6.

---

## Table of Contents

1. [Testing Framework Overview](#1-testing-framework-overview)
2. [Writing Test Prompts](#2-writing-test-prompts)
3. [Execution Testing: With-skill vs Baseline](#3-execution-testing-with-skill-vs-baseline)
4. [Quantitative Evaluation: Assertion-based Scoring](#4-quantitative-evaluation-assertion-based-scoring)
5. [Using Specialized Agents](#5-using-specialized-agents)
6. [Iterative Improvement Loop](#6-iterative-improvement-loop)
7. [Description Trigger Validation](#7-description-trigger-validation)
8. [Workspace Structure](#8-workspace-structure)

---

## 1. Testing Framework Overview

Skill quality validation is a combination of **qualitative evaluation** and **quantitative evaluation**.

| Evaluation Type | Method | Suitable Skills |
|----------------|--------|----------------|
| **Qualitative** | User directly reviews output | Subjective quality such as writing style, design, creative work |
| **Quantitative** | Assertion-based automated scoring | Objectively verifiable tasks such as file creation, data extraction, code generation |

Core loop: **Write → Run Tests → Evaluate → Improve → Retest**

---

## 2. Writing Test Prompts

### Principles

Test prompts must be **specific, natural sentences that a real user would actually type**. Abstract or artificial prompts have low test value.

### Bad Examples

```
"Process the PDF"
"Extract the data"
"Generate the chart"
```

### Good Examples

```
"In the file 'Q4_Sales_Final_v2.xlsx' in the Downloads folder, use column C (Revenue)
and column D (Cost) to add a profit margin (%) column. Then sort descending by profit margin."
```

```
"Extract the table on page 3 of this PDF and convert it to CSV. The table header is
two rows — the first row is the category, the second row is the actual column name."
```

### Prompt Diversity

- Mix **formal / casual** tone
- Mix **explicit / implicit** intent (cases where the file format is stated directly vs. must be inferred from context)
- Mix **simple / complex** tasks
- Some prompts should include abbreviations, typos, or casual phrasing

### Coverage

Start with 2–3 prompts, but design them to cover:
- 1 core use case
- 1 edge case
- (Optional) 1 compound task

---

## 3. Execution Testing: With-skill vs Baseline

### 3-1. Comparative Execution Structure

For each test prompt, spawn two subagents **simultaneously**:

**With-skill execution:**
```
prompt: "{test prompt}"
skill path: {skill path}
output path: _workspace/iteration-N/eval-{id}/with_skill/outputs/
```

**Baseline execution:**
```
prompt: "{test prompt}"  (same)
skill: none
output path: _workspace/iteration-N/eval-{id}/without_skill/outputs/
```

### 3-2. Baseline Selection

| Situation | Baseline |
|-----------|----------|
| Creating a new skill | Run same prompt without the skill |
| Improving an existing skill | Pre-modification skill version (preserve snapshot) |

### 3-3. Capturing Timing Data

Save `total_tokens` and `duration_ms` **immediately** from the subagent completion notification. This data is only accessible at notification time and cannot be recovered afterward.

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

---

## 4. Quantitative Evaluation: Assertion-based Scoring

### 4-1. Writing Assertions

When outputs are objectively verifiable, define assertions for automated scoring.

**Good assertions:**
- Objectively determinable as true/false
- Descriptive names that make it clear what is being checked just by reading the result
- Validate the core value of the skill

**Bad assertions:**
- Things that always pass regardless of whether the skill is used (e.g., "output exists")
- Things requiring subjective judgment (e.g., "well written")

### 4-2. Programmatic Verification

If an assertion can be verified with code, write it as a script. Faster and more reliable than visual inspection, and reusable across iterations.

### 4-3. Watch Out for Non-discriminating Assertions

Assertions that "pass 100% in both configurations" fail to measure the differential value of the skill. When you find such assertions, remove them or replace them with more challenging ones.

### 4-4. Scoring Result Schema

```json
{
  "expectations": [
    {
      "text": "Profit margin column added",
      "passed": true,
      "evidence": "Column 'profit_margin_pct' confirmed in column E"
    },
    {
      "text": "Sorted descending by profit margin",
      "passed": false,
      "evidence": "Original order retained without sorting"
    }
  ],
  "summary": {
    "passed": 1,
    "failed": 1,
    "total": 2,
    "pass_rate": 0.50
  }
}
```

---

## 5. Using Specialized Agents

Using agents in specialized roles during the test/evaluation process improves quality.

### 5-1. Grader

Performs assertion-based scoring, extracts verifiable claims from outputs, and cross-validates them.

**Role:**
- Pass/fail judgment per assertion + evidence
- Extract and verify factual claims from outputs
- Feedback on the quality of the eval itself (suggest improvements if assertions are too easy or ambiguous)

### 5-2. Comparator (Blind Comparator)

Anonymizes two outputs as A/B and judges quality without knowing which one used the skill.

**When to use:** When you want to rigorously verify "is the new version actually better?" Can be omitted in general iterative improvement.

**Judgment criteria:**
- Content: accuracy, completeness
- Structure: organization, formatting, usability
- Overall score

### 5-3. Analyzer

Analyzes statistical patterns in benchmark data:
- Non-discriminating assertions (both configurations pass → no discriminating power)
- High-variance evals (results vary significantly across runs → unstable)
- Time/token tradeoffs (skill improves quality but also raises cost)

---

## 6. Iterative Improvement Loop

### 6-1. Collecting Feedback

Show the output to the user and collect feedback. Treat empty feedback as "no issues."

### 6-2. Improvement Principles

1. **Generalize feedback** — Narrow fixes that only address the test example are overfitting. Fix at the principle level.
2. **Remove what doesn't earn its weight** — Read the transcript; if the skill is sending the agent on unproductive tasks, delete those parts.
3. **Explain the why** — Even if the user's feedback is brief, understand why it matters and incorporate that understanding into the skill.
4. **Bundle repetitive tasks** — If the same helper script is generated in every test run, include it in `scripts/` upfront.

### 6-3. Iteration Procedure

```
1. Modify the skill
2. Re-run all test cases in a new iteration-N+1/ directory
3. Present results to the user (compare with previous iteration)
4. Collect feedback
5. Modify again → repeat
```

**Exit conditions:**
- User is satisfied
- All feedback is empty (all outputs are issue-free)
- No more meaningful improvements to make

### 6-4. Draft → Review Pattern

When modifying a skill, write a draft, then **read it again with fresh eyes** and improve it. Do not try to write it perfectly in one pass — go through a draft-review cycle.

---

## 7. Description Trigger Validation

### 7-1. Writing Trigger Eval Queries

Write 20 eval queries — 10 should-trigger + 10 should-NOT-trigger.

**Query quality criteria:**
- Specific, natural sentences that a real user would actually type
- Include concrete details such as file paths, personal context, column names, company names
- Mix length, tone, and format
- Focus on **edge cases** rather than obvious correct answers

**Should-trigger queries (8–10):**
- Same intent expressed in different ways (formal/casual)
- Cases where the skill/file type is not stated explicitly but is clearly needed
- Minority use cases
- Cases that compete with other skills but this skill should win

**Should-NOT-trigger queries (8–10):**
- **Near-misses are key** — queries with similar keywords but where a different tool/skill is appropriate
- Clearly unrelated queries ("write a Fibonacci function") have no test value
- Adjacent domains, ambiguous phrasing, keyword overlap but different context

### 7-2. Checking for Conflicts with Existing Skills

Verify that the new skill's description does not overlap with the trigger territory of existing skills:

1. Collect descriptions from the list of existing skills
2. Verify that the new skill's should-trigger queries do not incorrectly trigger existing skills
3. If conflicts are found, describe boundary conditions more clearly in the description

### 7-3. Automated Optimization (Optional Advanced Feature)

When description optimization is needed:

1. Split 20 eval queries into Train (60%) / Test (40%)
2. Measure trigger accuracy with the current description
3. Analyze failure cases and generate an improved description
4. Select the best description based on the Test set (not the Train set — to prevent overfitting)
5. Repeat up to 5 times

> This process is performed by an automation script using `claude -p`. Token costs are high, so run it only after the skill is sufficiently stable, as a final step.

---

## 8. Workspace Structure

A directory structure for systematically managing test/evaluation results:

```
{skill-name}-workspace/
├── iteration-1/
│   ├── eval-descriptive-name-1/
│   │   ├── eval_metadata.json
│   │   ├── with_skill/
│   │   │   ├── outputs/
│   │   │   ├── timing.json
│   │   │   └── grading.json
│   │   └── without_skill/
│   │       ├── outputs/
│   │       ├── timing.json
│   │       └── grading.json
│   ├── eval-descriptive-name-2/
│   │   └── ...
│   └── benchmark.json
├── iteration-2/
│   └── ...
└── evals/
    └── evals.json
```

**Rules:**
- Use **descriptive names** for eval directories, not numbers (e.g., `eval-multi-page-table-extraction`)
- Preserve each iteration in an independent directory (do not overwrite previous iterations)
- Do not delete `_workspace/` — it is used for post-hoc verification and audit trail
