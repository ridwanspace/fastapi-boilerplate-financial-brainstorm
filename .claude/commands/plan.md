---
description: Research a feature or query, then generate a structured development plan saved to docs/plan/. Usage: /plan <feature description or question>
argument-hint: "<feature description, question, or filename>"
allowed-tools: Read, Glob, Grep, Bash
---

You are a senior software architect for this FastAPI financial project. Your job is to produce a thorough, implementation-ready development plan for the topic below.

**User query / topic**: $ARGUMENTS

---

## Step 0 — Load project context

Read `PRD.md` (if it exists) to understand:
- The chosen architecture and its folder structure conventions
- Financial design decisions and their implications
- Code conventions

If `PRD.md` does not exist, warn the user:
```
PRD.md not found. Consider running /brainstorm first to establish architecture decisions.
Proceeding with a generic plan — you may need to adapt the structure later.
```

Also read `.claude/CLAUDE.md` for project-level conventions.

---

## Step 1 — Understand the request

Parse `$ARGUMENTS`:
- If it looks like a filename (ends in `.md`, `.py`, etc.), read that file first for context.
- Otherwise treat it as a feature description or question.

Clarify the scope internally — you do not need to ask the user unless the request is genuinely ambiguous.

---

## Step 2 — Explore the codebase

Before writing anything, explore what already exists relevant to this topic.

Use Glob and Grep to find:
- Existing entities, models, services related to the topic
- Existing API routes and schemas
- Existing tests covering related areas
- Related config (e.g., `src/container.py`, router files, database base)

Read key files to understand current patterns and avoid re-inventing existing solutions.

---

## Step 3 — Determine the output file path(s) and plan split strategy

Find the next sequential plan file number:

```bash
ls docs/plan/ 2>/dev/null | sort | tail -1
```

- If the directory is empty → use `00`
- File name format: `{NN}-{kebab-case-topic}.md`

### When to split into multiple files

| Signal | Action |
|--------|--------|
| ≤ 3 phases **or** ≤ 12 tasks | Single file |
| 4–6 phases **or** 13–24 tasks | Split into **2** files |
| 7+ phases **or** 25+ tasks | Split into **3+** files |

---

## Step 4 — Write the plan(s)

Save each plan to `docs/plan/{NN}-{topic}.md` using this structure:

```markdown
# {Descriptive Title}

**Created**: {today's date YYYY-MM-DD}
**Updated**: {today's date YYYY-MM-DD}
**Priority**: (adjust 1–5 stars based on criticality)
**Status**: Planning
**Part**: {N of M} (omit for single-file plans)
**Depends On**: {prerequisites or "None"}
**Module**: {relevant module/feature name}

---

## Overview

{2–4 sentences describing what this plan covers and why.}

---

## Prerequisites

> **CRITICAL**: These must be complete before starting

**Required Infrastructure (All Ready)**:
- [x] {prerequisite 1}

**Environment**:
- [x] {required env var or service}

---

## Overall Progress

| Phase | Section | Total | Done | Status |
|-------|---------|-------|------|--------|
| P0 | {section name} | {N} | 0 | Not Started |
| **Total** | | **{N}** | **0** | **0%** |

---

## What Already Exists

| Entity / File | Location | Status |
|---------------|----------|--------|
| {name} | `{path}` | Exists / Partial / Missing |

---

## Module Structure

{Directory tree of files to create or modify.}

---

## Phase {N}: {Phase Name}

### Task {N}.{M}: {Task Name}

**File**: `{path/to/file.py}` (create / modify)

**Purpose**: {one sentence}

#### Implementation

{Key details, patterns to follow, code snippets.}

#### Acceptance Criteria

- [ ] {Requirement}

#### Tests

- [ ] `test_{what}_{expected}`: {description}

---

## Financial Safety Checklist

> Include if the feature touches money or transactions.
> Tailor to the financial decisions in PRD.md.

- [ ] {Relevant financial safety check based on PRD decisions}

---

## Architecture Compliance Checklist

> Tailor to the architecture chosen in PRD.md.

- [ ] {Architecture-specific compliance check}

---

## Testing Requirements Summary

- [ ] Unit tests: `pytest -m unit tests/unit/... -v`
- [ ] Integration tests: `pytest -m integration tests/integration/... -v`
- [ ] Coverage target: ≥ 80%

---

## Open Questions

{Unresolved decisions or edge cases.}
```

---

## Step 5 — Rules for writing a good plan

**Content rules**:
- "What Already Exists" must reflect actual codebase state — do not guess. Read files.
- Every task must have clear acceptance criteria and at least 2 test cases.
- Test names: `test_{what}_{expected_outcome}`
- Code snippets should match the project's actual patterns (from PRD.md or existing code).
- Financial safety checklist items should match the specific decisions in PRD.md, not be generic.

**Scope rules**:
- Do not implement code — this is a plan only.
- Do not modify any existing source files.
- Only create plan files in `docs/plan/`.

---

## Step 6 — Confirm to the user

After saving, report:
1. All file paths created
2. Count of tasks, unit tests, and integration tests
3. Any open questions to resolve before implementation
