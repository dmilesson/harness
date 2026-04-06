# Skill Writing Guide

A detailed writing guide for improving the quality of skills created in Harness. Supplementary reference for SKILL.md Phase 4.

---

## Table of Contents

1. [Description Writing Patterns](#1-description-writing-patterns)
2. [Body Writing Style](#2-body-writing-style)
3. [Output Format Definition Patterns](#3-output-format-definition-patterns)
4. [Example Writing Patterns](#4-example-writing-patterns)
5. [Progressive Disclosure Pattern](#5-progressive-disclosure-pattern)
6. [Script Bundling Decision Criteria](#6-script-bundling-decision-criteria)
7. [Data Schema Standards](#7-data-schema-standards)
8. [What Not to Include in Skills](#8-what-not-to-include-in-skills)

---

## 1. Description Writing Patterns

The description is the skill's only trigger mechanism. Claude decides whether to use a skill by looking only at the name + description in the `available_skills` list.

### Understanding the Trigger Mechanism

Claude tends not to invoke skills for simple tasks it can handle easily with its built-in tools. A simple request like "read this PDF for me" may not trigger even with a perfect description. The more complex, multi-step, and specialized a task is, the higher the probability of triggering a skill.

### Writing Principles

1. Describe both **what the skill does** and **the specific trigger situations**
2. Specify boundary conditions that distinguish similar cases that should NOT trigger
3. Be slightly "pushy" — compensate for Claude's tendency to be conservative about triggering

### Good Examples

```yaml
description: "Handles all PDF tasks: reading PDF files, extracting text/tables,
  merging, splitting, rotating, watermarking, encrypting/decrypting, OCR, and more.
  When a .pdf file is mentioned or a PDF output is requested, always use this skill.
  Especially useful when transformation/editing/analysis is needed, not just
  'reading' a PDF."
```

```yaml
description: "All spreadsheet tasks including adding columns, formula calculations,
  formatting, charts, and data cleaning for Excel/CSV/TSV files. When the user
  mentions a spreadsheet file — even casually ('the xlsx in my downloads folder') —
  use this skill."
```

### Bad Examples

- `"A skill that processes data"` — too vague, unclear what file/task is involved
- `"PDF-related tasks"` — no list of specific operations, trigger situations not described

---

## 2. Body Writing Style

### Why-First Principle

When an LLM understands the reason, it makes correct judgments even in edge cases. Conveying context is more effective than imposing rigid rules.

**Bad example:**
```markdown
ALWAYS use pdfplumber for table extraction. NEVER use PyPDF2 for tables.
```

**Good example:**
```markdown
Use pdfplumber for table extraction. PyPDF2 is specialized for text extraction
and cannot preserve the row/column structure of tables. pdfplumber recognizes
cell boundaries and returns structured data.
```

### Generalization Principle

When a problem is found in feedback or test results, instead of a narrow fix that only addresses the specific example, **generalize to the principle level**.

**Overfitting fix:**
```markdown
If there is a "Q4 Revenue" column, convert that column to a number.
```

**Generalized fix:**
```markdown
If a column name contains keywords implying numeric values such as "revenue",
"amount", or "quantity", convert that column to a numeric type.
Retain the original value if conversion fails.
```

### Imperative Tone

Use imperative forms like "do this", "use this" rather than "you can do this" or "it is possible to". Skills are instructions.

### Context Economy

The context window is a shared resource. Ask whether every sentence justifies its token cost:
- "Does Claude already know this?" → delete
- "Will Claude make a mistake without this explanation?" → keep
- "Is one concrete example more effective than a long explanation?" → replace with an example

---

## 3. Output Format Definition Patterns

Use in skills where the format of the output matters:

```markdown
## Report Structure
Follow this template exactly:

# [Title]
## Summary
## Key Findings
## Recommendations
```

Keep format definitions concise; including a real example makes them even more effective.

---

## 4. Example Writing Patterns

Examples are more effective than long explanations:

```markdown
## Commit Message Format

**Example 1:**
Input: Add JWT token-based user authentication
Output: feat(auth): implement JWT-based authentication

**Example 2:**
Input: Fix bug where show password button on login page does not work
Output: fix(login): fix password visibility toggle button behavior
```

---

## 5. Progressive Disclosure Pattern

### Pattern 1: Domain Separation

```
bigquery-skill/
├── SKILL.md (overview + domain selection guide)
└── references/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    └── product.md (API usage, features)
```

When the user asks about revenue, load only finance.md.

### Pattern 2: Conditional Detail

```markdown
# DOCX Processing

## Document Creation
Create new documents with docx-js. → See [DOCX-JS.md](references/docx-js.md).

## Document Editing
For simple edits, modify the XML directly.
**If tracked changes are needed**: see [REDLINING.md](references/redlining.md)
```

### Pattern 3: Large Reference File Structure

Reference files over 300 lines should include a table of contents at the top:

```markdown
# API Reference

## Table of Contents
1. [Authentication](#authentication)
2. [Endpoint List](#endpoint-list)
3. [Error Codes](#error-codes)
4. [Rate Limits](#rate-limits)

---

## Authentication
...
```

---

## 6. Script Bundling Decision Criteria

Observe agent transcripts during test runs. Bundle when the following patterns appear:

| Signal | Action |
|--------|--------|
| Same helper script generated in all 3 of 3 tests | Bundle into `scripts/` |
| Same pip install/npm install runs every time | Explicitly include dependency installation step in skill |
| Same multi-step approach repeated every time | Describe as standard procedure in skill body |
| Same error followed by the same workaround every time | Document known issues and solutions in skill |

Bundled scripts must pass an execution test before being included.

---

## 7. Data Schema Standards

Use standard schemas for consistency in data exchange between skills. These can be used for testing and evaluation of skills created in Harness.

### eval_metadata.json

Metadata for each test case:

```json
{
  "eval_id": 0,
  "eval_name": "descriptive-name-here",
  "prompt": "The user's task prompt",
  "assertions": [
    "Output contains X",
    "File was created in Y format"
  ]
}
```

### grading.json

Assertion-based grading results:

```json
{
  "expectations": [
    {
      "text": "Output contains 'Seoul'",
      "passed": true,
      "evidence": "Confirmed 'Extract Seoul regional data' in step 3"
    }
  ],
  "summary": {
    "passed": 2,
    "failed": 1,
    "total": 3,
    "pass_rate": 0.67
  }
}
```

**Field name note:** Use exactly `text`, `passed`, `evidence` — do not use variants like `name`/`met`/`details`.

### timing.json

Execution time/token measurements:

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

Save `total_tokens` and `duration_ms` immediately from the subagent completion notification. This data is only accessible at the time of notification and cannot be recovered later.

---

## 8. What Not to Include in Skills

- Supplementary documentation such as README.md, CHANGELOG.md, INSTALLATION_GUIDE.md
- Meta-information from the skill creation process (test results, iteration history)
- End-user documentation (skills are instructions for AI agents)
- General knowledge that Claude already knows
