# Superpowers Skills → oh-my-opencode Subagents Integration Guide

> **Goal**: When Superpowers Skills need to dispatch sub-roles, automatically route to oh-my-opencode's corresponding Agents  
> **Prerequisites**: You're running in an OpenCode + oh-my-opencode plugin environment

---

## 1. Technical Integration Points Analysis

### 1.1 OmO's Two Sub-Agent Dispatch Methods

oh-my-opencode provides **two tools** for dispatching sub-Agents:

```
Method A: task() tool — dispatch via category
┌────────────────────────────────────────────────────────┐
│ task(                                                  │
│   category="quick",          ← one of 8 built-in categories │
│   load_skills=["tdd"],       ← inject skills               │
│   description="Fix type error",                        │
│   prompt="...",                                        │
│   run_in_background=false     ← sync/async              │
│ )                                                      │
│ → Routes to Sisyphus-Junior + category-specific model  │
└────────────────────────────────────────────────────────┘

Method B: task() tool — dispatch via subagent_type
┌────────────────────────────────────────────────────────┐
│ task(                                                  │
│   subagent_type="oracle",     ← specify Agent name     │
│   description="Architecture review",                   │
│   prompt="...",                                        │
│   run_in_background=true      ← usually background     │
│ )                                                      │
│ → Directly routes to Oracle Agent + its fallback chain │
└────────────────────────────────────────────────────────┘

Method C: call_omo_agent() tool — explore/librarian only
┌────────────────────────────────────────────────────────┐
│ call_omo_agent(                                        │
│   subagent_type="explore",                             │
│   description="Find auth patterns",                    │
│   prompt="...",                                        │
│   run_in_background=true                               │
│ )                                                      │
│ → Only allows explore and librarian                    │
└────────────────────────────────────────────────────────┘
```

### 1.2 SP's Sub-Role Dispatch Method

Superpowers' Skills "dispatch" sub-roles through the following methods:

```
Method A: Task tool + prompt template (Claude Code environment)
  task(description="Implement Task N", prompt="<implementer-prompt.md template content>")

Method B: Instructions in Skill text (all environments)
  "Dispatch superpowers:code-reviewer subagent"
  → Actually has main Agent read agents/code-reviewer.md and work in that role
```

### 1.3 Core Integration Problem

Superpowers says "dispatch an Implementer", but:
- It doesn't know OmO has a `category` system
- It doesn't know OmO has a `subagent_type` parameter
- It doesn't know OmO has a `load_skills` injection mechanism

**Solution: Modify SP's Skill files so their role dispatch instructions use OmO's task() API format.**

---

## 2. Specific Mapping Rules

| SP Role | Corresponding OmO Call | Parameters |
|---------|----------------------|------------|
| **Controller** (main) | No mapping needed, Sisyphus itself is the Controller | — |
| **Implementer** (worker) | `task(category="quick")` or `task(category="deep")` | Choose category by task complexity |
| **Spec Reviewer** (spec review) | `task(subagent_type="momus")` | Review target needs to change from "code vs. spec" to "implementation vs. spec" |
| **Code Reviewer** (code review) | `task(subagent_type="oracle")` | Oracle excels at architecture and deep review |
| **brainstorming** | `/start-work` or Sisyphus directly calls Prometheus | — |
| **writing-plans** | Prometheus (via OmO's `/start-work` command) | — |

---

## 3. Files Requiring Modification

```
superpowers/
├── skills/
│   ├── subagent-driven-development/
│   │   ├── SKILL.md                        ← Modify: replace dispatch instructions
│   │   ├── implementer-prompt.md           ← Modify: use OmO task() format
│   │   ├── spec-reviewer-prompt.md         ← Modify: specify subagent_type="momus"
│   │   └── code-quality-reviewer-prompt.md ← Modify: specify subagent_type="oracle"
│   ├── executing-plans/
│   │   └── SKILL.md                        ← Modify: replace dispatch instructions
│   ├── dispatching-parallel-agents/
│   │   └── SKILL.md                        ← Modify: use OmO's background mode
│   ├── requesting-code-review/
│   │   └── SKILL.md                        ← Modify: use subagent_type="oracle"
│   ├── writing-plans/
│   │   └── SKILL.md                        ← Modify: specify using Prometheus
│   └── brainstorming/
│       └── SKILL.md                        ← Modify: specify using Metis + Prometheus
└── agents/
    └── code-reviewer.md                    ← Optional: merge into Oracle prompt
```

---

## 4. File-by-File Modification Plan

### 4.1 `subagent-driven-development/SKILL.md` — Controller Process

**Location to modify**: The entire "Dispatch" process description, replacing "Task tool (general-purpose)" with OmO's `task()` API.

**Original** (around lines 127-131):
```
[Dispatch implementation subagent with full task text + context]
```

**Change to**:

```markdown
## Dispatch Instructions

### Implementer (per task)

Determine task complexity and choose the appropriate category:

| Task Type | Category | When |
|-----------|----------|------|
| 1-2 files, complete spec | `quick` | Most implementation tasks with clear specs |
| Multiple files, integration concerns | `deep` | Cross-module work, requires exploration |
| Hard logic, architecture decisions | `ultrabrain` | Complex algorithmic or design work |
| Frontend, UI/UX | `visual-engineering` | Any visual/design work |

Dispatch format:
```
task(
  category="[chosen-category]",
  load_skills=["test-driven-development"],
  description="Implement Task N: [task name]",
  prompt="[FULL task text + context from plan]",
  run_in_background=false
)
```

**CRITICAL**: Always include `load_skills=["test-driven-development"]` for implementation tasks.

### Spec Compliance Reviewer

After implementer reports DONE, dispatch spec review:

```
task(
  subagent_type="momus",
  description="Spec compliance review for Task N",
  prompt="Review the implementation against its specification.

## What Was Requested
[FULL TEXT of task requirements]

## What Was Implemented
[From implementer's report]

Verify by reading actual code, NOT by trusting the report.
Report: [OKAY] if compliant, or [REJECT] with specific issues.",
  run_in_background=false
)
```

### Code Quality Reviewer

After spec compliance passes:

```
task(
  subagent_type="oracle",
  description="Code quality review for Task N",
  prompt="Review the code quality of the implementation.

## What Was Implemented
[From implementer's report]

## Review Scope
- Code quality, naming, organization
- Test coverage and quality
- Architecture adherence
- Potential issues

BASE_SHA: [commit before task]
HEAD_SHA: [current commit]

Return: Strengths, Issues (Critical/Important/Minor), Assessment",
  run_in_background=false
)
```
```

### 4.2 `subagent-driven-development/implementer-prompt.md` — Implementer Template

**Original**: Uses generic `Task tool (general-purpose)` format.

**Change to**:

```markdown
# Implementer Subagent Prompt Template

Use this template when dispatching an implementer subagent via oh-my-opencode.

```
task(
  category="[quick|deep|ultrabrain|visual-engineering]",
  load_skills=["test-driven-development"],
  description="Implement Task N: [task name]",
  prompt: "You are implementing Task N: [task name]

## Task Description

[FULL TEXT of task from plan - paste it here, don't make subagent read file]

## Context

[Scene-setting: where this fits, dependencies, architectural context]

## Before You Begin

[... Keep original questioning/clarification instructions unchanged ...]

## Your Job

Once you're clear on requirements:
1. Implement exactly what the task specifies
2. Write tests (following TDD)
3. Verify implementation works
4. Commit your work
5. Self-review (see below)
6. Report back

Work from: [directory]

## Report Format

When done, report:
- **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
- What you implemented (or what you attempted, if blocked)
- What you tested and test results
- Files changed
- Self-review findings (if any)
- Any issues or concerns",
  run_in_background=false
)
```

### Category Selection Guide

| Task Complexity | Category | Model |
|----------------|----------|-------|
| 1-2 files, clear spec | `quick` | gpt-5.4-mini (fast) |
| Multi-file, integration | `deep` | gpt-5.4 medium (autonomous) |
| Hard logic, architecture | `ultrabrain` | gpt-5.4 xhigh (reasoning) |
| Frontend, UI/UX | `visual-engineering` | gemini-3.1-pro high |

**Always load `test-driven-development` skill for implementation tasks.**
```

### 4.3 `subagent-driven-development/spec-reviewer-prompt.md` → Momus

**Change to**:

```markdown
# Spec Compliance Reviewer Prompt Template

Dispatch via oh-my-opencode's task tool targeting Momus agent:

```
task(
  subagent_type="momus",
  description="Spec compliance review for Task N",
  prompt="You are reviewing whether an implementation matches its specification.

## What Was Requested

[FULL TEXT of task requirements]

## What Implementer Claims They Built

[From implementer's report]

## CRITICAL: Do Not Trust the Report

The implementer finished suspiciously quickly. Their report may be incomplete,
inaccurate, or optimistic. You MUST verify everything independently.

**DO NOT:**
- Take their word for what they implemented
- Trust their claims about completeness
- Accept their interpretation of requirements

**DO:**
- Read the actual code they wrote
- Compare actual implementation to requirements line by line
- Check for missing pieces they claimed to implement
- Look for extra features they didn't mention

Report:
- **[OKAY]** if spec compliant (everything matches after code inspection)
- **[REJECT]** with specific issues: [list what's missing or extra, with file:line references]",
  run_in_background=false
)
```
```

### 4.4 `subagent-driven-development/code-quality-reviewer-prompt.md` → Oracle

**Change to**:

```markdown
# Code Quality Reviewer Prompt Template

Dispatch via oh-my-opencode's task tool targeting Oracle agent:

```
task(
  subagent_type="oracle",
  description="Code quality review for Task N: [task name]",
  prompt="Review the code quality of a completed implementation.

## What Was Implemented
[From implementer's report]

## Review Focus
1. Code quality: naming, organization, maintainability
2. Test coverage: are tests verifying real behavior?
3. Architecture: follows existing patterns? SOLID?
4. Issues: Critical / Important / Minor

BASE_SHA: [commit before task]
HEAD_SHA: [current commit]

Return:
- **Bottom line**: 2-3 sentence assessment
- **Strengths**: What's done well
- **Issues**: Categorized by severity with specific file:line references
- **Assessment**: Ready to proceed / Needs fixes",
  run_in_background=false
)
```
```

### 4.5 `dispatching-parallel-agents/SKILL.md` — Parallel Dispatch

**Original**: Uses `Task("Fix ...")` format.

**Modify key section** (around lines 66-78):

```markdown
### 3. Dispatch in Parallel

Use oh-my-opencode's task tool with background mode:

```
// Parallel background dispatch
task(category="quick", load_skills=[], description="Fix agent-tool-abort.test.ts", prompt="...", run_in_background=true)
task(category="quick", load_skills=[], description="Fix batch-completion.test.ts", prompt="...", run_in_background=true)
task(category="quick", load_skills=[], description="Fix tool-approval.test.ts", prompt="...", run_in_background=true)
// All three run concurrently
```

Or for research/exploration tasks, use specialized agents:

```
task(subagent_type="explore", description="Find auth patterns", prompt="...", run_in_background=true)
task(subagent_type="librarian", description="Research JWT best practices", prompt="...", run_in_background=true)
```

### 4. Review and Integrate

Collect results with:
```
background_output(task_id="bg_task_1")
background_output(task_id="bg_task_2")
background_output(task_id="bg_task_3")
```
```

### 4.6 `writing-plans/SKILL.md` — Plan Writing

**Add OmO routing info in Execution Handoff section**:

```markdown
## Execution Handoff (oh-my-opencode integrated)

After saving the plan, offer execution choice:

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**

**1. OmO Subagent-Driven (recommended)**
Dispatch each task to oh-my-opencode's category-based agents:

```
task(category="quick", load_skills=["test-driven-development"], description="Task 1: ...", prompt="...", run_in_background=false)
```

After each task completes, dispatch review:
```
task(subagent_type="momus", description="Review Task 1", prompt="...", run_in_background=false)
task(subagent_type="oracle", description="Quality review Task 1", prompt="...", run_in_background=false)
```

**2. OmO Parallel Waves**
For independent tasks, dispatch by wave using background mode:

```
// Wave 1 - all independent tasks in parallel
task(category="quick", load_skills=["tdd"], description="Task 1", prompt="...", run_in_background=true)
task(category="deep", load_skills=["tdd"], description="Task 5", prompt="...", run_in_background=true)
```
```

### 4.7 `brainstorming/SKILL.md` — Requirements Analysis

**Add OmO routing in Implementation section**:

```markdown
## Implementation (oh-my-opencode integrated)

- Invoke Prometheus for planning:
  ```
  /start-work
  ```
  Or directly:
  ```
  task(subagent_type="prometheus", description="Create plan for [feature]", prompt="...", run_in_background=false)
  ```

- Before planning complex tasks, consult Metis for pre-analysis:
  ```
  task(subagent_type="metis", description="Analyze requirements for [feature]", prompt="...", run_in_background=false)
  ```
```

---

## 5. Advanced: Create Custom Categories for SP Role Mapping

If you don't want to specify `subagent_type` in prompts every time, you can create **custom Categories** in OmO config to map SP roles:

### `~/.config/opencode/oh-my-opencode.jsonc`

```jsonc
{
  // Custom categories mapping Superpowers roles
  "categories": {
    // SP Implementer (simple) → fast model
    "sp-implementer": {
      "model": "openai/gpt-5.4-mini",
      "description": "Superpowers Implementer - well-specified single tasks"
    },
    // SP Implementer (complex) → deep model
    "sp-implementer-deep": {
      "model": "openai/gpt-5.4",
      "variant": "medium",
      "description": "Superpowers Implementer - complex multi-file tasks"
    },
    // SP Spec Reviewer → review model
    "sp-spec-reviewer": {
      "model": "openai/gpt-5.4",
      "variant": "xhigh",
      "description": "Spec compliance review"
    },
    // SP Code Reviewer → reasoning model
    "sp-code-reviewer": {
      "model": "openai/gpt-5.4",
      "variant": "xhigh",
      "description": "Code quality review"
    }
  }
}
```

Then Skills can use:

```
task(category="sp-implementer", load_skills=["test-driven-development"], ...)
task(category="sp-spec-reviewer", load_skills=[], ...)
task(category="sp-code-reviewer", load_skills=[], ...)
```

---

## 6. Maximum Advanced: Create a Bridge Skill for Automatic Mapping

Create a new OmO Skill to automatically bridge:

### `.opencode/skills/sp-omo-bridge/SKILL.md`

```yaml
---
name: sp-omo-bridge
description: Bridge between Superpowers workflow roles and oh-my-opencode agents. Automatically maps SP roles to OMO agents.
---
```

```markdown
# SP → OMO Agent Bridge

When Superpowers workflow dispatches a sub-agent role, use this mapping:

## Role → Agent Mapping

| SP Role | OMO Tool | Parameters |
|---------|----------|------------|
| Implementer (simple) | `task()` | `category="quick", load_skills=["test-driven-development"]` |
| Implementer (complex) | `task()` | `category="deep", load_skills=["test-driven-development"]` |
| Implementer (frontend) | `task()` | `category="visual-engineering", load_skills=["test-driven-development", "frontend-ui-ux"]` |
| Spec Reviewer | `task()` | `subagent_type="momus"` |
| Code Quality Reviewer | `task()` | `subagent_type="oracle"` |
| Codebase Search | `task()` | `subagent_type="explore", run_in_background=true` |
| External Research | `task()` | `subagent_type="librarian", run_in_background=true` |
| Pre-planning Analysis | `task()` | `subagent_type="metis"` |
| Plan Review | `task()` | `subagent_type="momus"` |

## Automatic Skill Injection

When dispatching implementation tasks, ALWAYS inject relevant skills:

| Task Type | load_skills |
|-----------|-------------|
| Any implementation | `["test-driven-development"]` |
| Bug fix | `["test-driven-development", "systematic-debugging"]` |
| Feature work | `["test-driven-development", "verification-before-completion"]` |
| Frontend | `["test-driven-development", "frontend-ui-ux"]` |
| Git operations | `["git-master"]` |

## Two-Stage Review (Superpowers Discipline)

When executing subagent-driven-development workflow:

1. After implementer DONE → dispatch `subagent_type="momus"` with spec compliance prompt
2. After spec OKAY → dispatch `subagent_type="oracle"` with code quality prompt
3. If either review finds issues → continue implementer session with `session_id`
4. Never skip either review stage
```

---

## 7. Summary: Three Integration Paths

| Approach | Change Volume | Effect | Recommended Scenario |
|----------|--------------|--------|---------------------|
| **A. Modify SP Skill Files** | Medium (modify 6-7 files) | SP workflow directly uses OmO Agents | Long-term use, deep integration |
| **B. Add OmO Custom Categories** | Small (modify 1 config file) | Maps roles via category names | Quick prototyping, flexible adjustment |
| **C. Create Bridge Skill** | Small (add 1 new Skill) | Declarative mapping table, Sisyphus auto-reads | Minimal intrusion, adjustable anytime |

**Recommendation**: Start with **C (Bridge Skill)**, verify results, then do **A (Modify SP Files)** for deep integration. If you want runtime configuration flexibility, add **B (Custom Categories)**.

Core principle: **Superpowers defines "what role to act as", oh-my-opencode provides "what model/tools to use".** You just need to build a bridge between SP's role dispatch points and OmO's Agent entry points.
