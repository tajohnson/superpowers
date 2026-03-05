---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).

**Save plans to:** `docs/plans/YYYY-MM-DD-<feature-name>.md`

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

**Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

**Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

**Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

**Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## Remember
- Exact file paths always
- Complete code in plan (not "add validation")
- Exact commands with expected output
- Reference relevant skills with @ syntax
- DRY, YAGNI, TDD, frequent commits

## External Plan Review

After saving the plan file, **before offering execution options**, run an external review if a reviewer CLI is available.

### Step 1: Check for reviewer

Check if a reviewer CLI is installed. The plugin config file at `${CLAUDE_PLUGIN_ROOT}/config/plan-review.json` specifies which adapter to use. If the config file doesn't exist, skip review and proceed to Execution Handoff.

### Step 2: Run the review

Use the Bash tool to run the review script:

```bash
bash "${CLAUDE_PLUGIN_ROOT}/hooks/plan-review" "<path-to-plan-file>"
```

This script sends the plan to an external AI CLI (Codex or Gemini) and prints review feedback to stdout.

### Step 3: Handle review result

- If the output starts with **"LGTM"**: The plan passed review. Announce the result briefly and proceed to Execution Handoff.
- If the output contains **actionable feedback**: Read the feedback carefully, revise the plan to address the substantive issues, save the updated plan file, and re-run the review (Step 2). The second review verifies that the revision actually addressed the feedback. Do not revise more than `maxReviews` times (default 2, configurable in `${CLAUDE_PLUGIN_ROOT}/config/plan-review.json`). After the limit, proceed to Execution Handoff regardless and note any unresolved feedback for the user.
- If the **script exits with an error** or the reviewer CLI is not found: Announce that external review was skipped (reviewer not available) and proceed to Execution Handoff.

### Important

- Do NOT enter or exit plan mode during review. This is a normal-mode Bash call.
- The review is advisory. The user makes the final call on whether to proceed.
- Always show the user a summary of what the reviewer said, whether LGTM or feedback.

## Execution Handoff

After saving the plan (and completing external review if configured), offer execution choice:

**"Plan complete and saved to `docs/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (this session)** - I dispatch fresh subagent per task, review between tasks, fast iteration

**2. Parallel Session (separate)** - Open new session with executing-plans, batch execution with checkpoints

**Which approach?"**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Stay in this session
- Fresh subagent per task + code review

**If Parallel Session chosen:**
- Guide them to open new session in worktree
- **REQUIRED SUB-SKILL:** New session uses superpowers:executing-plans
