# Superpowers Project Architecture Analysis Report

> **Project**: obra/superpowers v5.0.7  
> **Positioning**: Complete software development workflow system for AI coding assistants  
> **Author**: Jesse Vincent (Prime Radiant)  
> **License**: MIT  
> **Analysis Date**: 2026-04-19

---

## 1. Project Overview

Superpowers is a **workflow skills framework** designed for AI coding assistants (Claude Code, Gemini CLI, Cursor, Codex, Copilot CLI). It does not provide low-level tools or Agent orchestration capabilities, but instead constrains and guides AI Agent behavior through a carefully designed set of **Skill** files — covering the complete software development lifecycle from brainstorming, design review, plan writing to test-driven development, debugging, and code review.

Core philosophy: **Encode software engineering discipline as AI-executable instructions**.

### Project Scale

| Metric | Value |
|--------|-------|
| Skills | 14 |
| Agents | 1 (code-reviewer) |
| Hooks | 1 (SessionStart) |
| Commands | 3 |
| Code Language | Primarily Markdown (SKILL.md files) |
| Runtime Dependencies | Minimal (package.json only 6 lines) |

---

## 2. Core Architecture

### 2.1 Overall Architecture Diagram

```
┌─────────────────────────────────────────────────────┐
│              AI Coding Assistant                      │
│  (Claude Code / Gemini CLI / Cursor / Codex / ...)    │
└─────────────┬───────────────────────────────────────┘
              │ Plugin System / Extension
              ▼
┌─────────────────────────────────────────────────────┐
│                 Superpowers Plugin                    │
│                                                       │
│  ┌──────────┐  ┌──────────┐  ┌───────────────────┐  │
│  │  Skills   │  │  Agents  │  │     Commands      │  │
│  │ (14)      │  │ (1)      │  │   (3)             │  │
│  └──────────┘  └──────────┘  └───────────────────┘  │
│                                                       │
│  ┌──────────┐  ┌──────────────────────────────────┐  │
│  │  Hooks   │  │    GEMINI.md / CLAUDE.md          │  │
│  │ (1)      │  │    (Platform entry files)          │  │
│  └──────────┘  └──────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

### 2.2 Technical Implementation Characteristics

**Extremely Lightweight**: The entire project's `package.json` is only 6 lines, with no JS/TS source code. All "logic" is written in Markdown files (SKILL.md), read and executed directly by AI Agents. This is a **Prompt-as-Code** architectural paradigm.

---

## 3. Core Mechanism Details

### 3.1 Skills System — The Heart of the Project

Skill is the core abstraction of Superpowers. Each Skill is a **Markdown folder** containing a `SKILL.md` main file and optional auxiliary files.

#### Skill Format Specification

```yaml
---
name: skill-name                          # Skill name
description: Trigger conditions and purpose description
---

# Skill Title

## Complete instruction content
(processes, checklists, anti-patterns, flowcharts, etc.)
```

#### Complete Overview of 14 Skills

| Category | Skill | Purpose | Type |
|----------|-------|---------|------|
| **Process Entry** | `using-superpowers` | Auto-loaded at session start, establishes skill discovery mechanism | Meta-skill |
| **Design** | `brainstorming` | Socratic-style design refinement, forces design before coding | Rigid process |
| **Planning** | `writing-plans` | Breaks design into 2-5 minute atomic tasks | Rigid process |
| **Execution** | `subagent-driven-development` | Sub-agent driven development, two-stage review | Rigid process |
| **Execution** | `executing-plans` | Batch plan execution with checkpoints | Rigid process |
| **Execution** | `dispatching-parallel-agents` | Parallel dispatch of independent tasks | Flexible pattern |
| **Quality** | `test-driven-development` | RED-GREEN-REFACTOR cycle | Rigid process |
| **Quality** | `systematic-debugging` | 4-stage root cause analysis | Rigid process |
| **Quality** | `verification-before-completion` | Mandatory verification before completion | Rigid process |
| **Collaboration** | `requesting-code-review` | Pre-code review checklist | Flexible pattern |
| **Collaboration** | `receiving-code-review` | Responding to review feedback | Flexible pattern |
| **Git** | `using-git-worktrees` | Isolated workspace management | Flexible pattern |
| **Git** | `finishing-a-development-branch` | Branch completion decision flow | Flexible pattern |
| **Meta** | `writing-skills` | Guide for creating new Skills | Flexible pattern |

#### Skill Trigger Mechanism

Skills are not passive reference documents but **actively triggered workflows**:

1. `using-superpowers` is auto-loaded at session start
2. It forces the Agent to check for applicable Skills before any response
3. Trigger rule: Even if there's only a 1% chance of relevance, it must be invoked
4. Skills are divided into **rigid** (must be strictly followed) and **flexible** (adaptable to context)

#### Skill Lifecycle (Standard Development Workflow)

```
brainstorming → writing-plans → subagent-driven-development / executing-plans
                                        │
                            ┌───────────┴───────────┐
                            │   Inner loop per task:  │
                            │   TDD → Debug → Verify │
                            └───────────┬───────────┘
                                        │
                            finishing-a-development-branch
```

### 3.2 Agents System

Superpowers only defines **1 Agent**: `code-reviewer`.

This is not an executable Agent implementation, but a **Prompt definition file** (`agents/code-reviewer.md`) describing the code reviewer's role, responsibilities, and review process. It is injected into the AI Agent's context for use.

Review dimensions include:
1. Plan alignment analysis (implementation vs. plan)
2. Code quality assessment
3. Architecture and design review
4. Documentation and standards
5. Issue grading (Critical / Important / Suggestion)

### 3.3 Hooks System

`hooks/hooks.json` defines a `SessionStart` hook that matches `startup|clear|compact` events and executes the `run-hook.cmd session-start` command.

This is the only Hook in the entire project — **minimalist design**, only triggered at session start to ensure the Skills discovery mechanism is injected into the Agent context.

### 3.4 Commands System

Three commands (defined in the `commands/` directory):

| Command | File | Purpose |
|---------|------|---------|
| `/brainstorm` | `brainstorm.md` | Directly triggers the brainstorming process |
| `/write-plan` | `write-plan.md` | Directly triggers plan writing |
| `/execute-plan` | `execute-plan.md` | Directly triggers plan execution |

These commands are shortcut entries for Skills, allowing users to manually trigger specific workflows.

---

## 4. Design Philosophy Deep Analysis

### 4.1 "Prompt-as-Code" Paradigm

Superpowers' most unique architectural decision: **not writing a single line of executable code**. The entire system consists of Markdown documents, with AI Agents acting as "interpreters" executing these "Prompt programs".

This means:
- **Zero runtime dependencies**: No need for Node.js / Python / any runtime
- **Natural cross-platform compatibility**: Any AI Agent that can read Markdown can use it
- **Human-readable and auditable**: Every Skill is clear documentation
- **Modify and it takes effect**: Changing SKILL.md is equivalent to changing code

### 4.2 Discipline-First

Every Skill has built-in strong **anti-pattern detection** and **rationalization prevention** mechanisms:

- **Red Flags tables**: Lists "lazy" thoughts an Agent might have and their reality
- **Rationalization Prevention**: Prevents Agents from making excuses to skip processes
- **Iron Law**: E.g., "NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST"
- **Hard Gate**: E.g., in brainstorming, "Do NOT invoke any implementation skill until design is approved"

### 4.3 Workflow Orchestration Pattern

Superpowers uses a **Pipeline pattern** to orchestrate Skills:

```
brainstorming → writing-plans → execution → review → finish
```

Each stage's **terminal state** clearly points to the next Skill (not arbitrary jumps), forming a deterministic workflow. This design ensures the Agent doesn't skip critical steps.

### 4.4 Sub-Agent Driven Development

`subagent-driven-development` is the most complex Skill, implementing:

1. **Controller-Worker pattern**: The main Agent is the Controller, each task dispatches a new Worker
2. **Two-stage review**: First spec compliance review, then code quality review
3. **Context isolation**: Each Worker receives precisely constructed context, not inheriting main session history
4. **Model selection strategy**: Chooses models of different capabilities based on task complexity (saving costs)

---

## 5. Multi-Platform Adaptation

Superpowers adapts to multiple platforms through different entry files:

| Platform | Entry | Mechanism |
|----------|-------|-----------|
| Claude Code | Plugin Marketplace | `GEMINI.md` + `hooks.json` |
| Gemini CLI | `GEMINI.md` | `@./skills/...` file references |
| Cursor | Plugin Marketplace | Same as Claude Code |
| Codex | INSTALL.md | URL instruction installation |
| OpenCode | INSTALL.md | URL instruction installation |
| Copilot CLI | Marketplace | Standard plugin mechanism |

`GEMINI.md` uses `@./skills/using-superpowers/SKILL.md` syntax to reference the main entry Skill.

---

## 6. Testing Strategy

Superpowers itself does not contain executable test code. Its "testing" is embodied in:

1. **Built-in Skill verification**: Each Skill contains verification checklists
2. **Self-Review mechanism**: Automatic self-review after plan writing
3. **Two-stage review**: Spec + quality review in subagent-driven-development

---

## 7. Strengths and Limitations

### Strengths

1. **Extremely lightweight**: No runtime, no dependencies, no build steps
2. **Cross-platform compatible**: Prompt-as-Code naturally adapts to all AI platforms
3. **Complete workflow**: Covers the full lifecycle from design to delivery
4. **Strong discipline**: Enforces engineering practices through Red Flags and Iron Laws
5. **Extensible**: Anyone can create new Skills following the `writing-skills` specification

### Limitations

1. **No tool layer**: Does not provide LSP, AST, file operations and other low-level tools
2. **No Agent orchestration**: Depends on host platform's Agent scheduling capabilities
3. **No state management**: Skills are stateless pure text, not tracking execution progress
4. **Platform differences**: Different platforms' capability differences may cause inconsistent Skill execution
5. **Single Agent**: Only one code-reviewer Agent definition
