# Superpowers vs oh-my-opencode: Agent Role Mapping Analysis

> **Analysis Date**: 2026-04-19  
> **Versions**: superpowers v5.0.7 / oh-my-opencode v3.17.4

---

## 0. Key Premise: Differences in Agent Concepts

These two frameworks have completely different definitions of "Agent", which is the foundation for understanding the mapping relationship:

| Dimension | Superpowers | oh-my-opencode |
|-----------|-------------|----------------|
| **Agent Definition** | A Markdown file describing a role Prompt | A TypeScript factory function creating a runtime Agent |
| **Agent Count** | 1 formal Agent (`code-reviewer`) | 11 formal Agents |
| **Implicit Agents** | 3 sub-roles defined via Prompt Templates within Skills | No implicit Agents, all formally registered |
| **Execution Method** | Dispatched by host platform's Task tool, Skills guide behavior via Prompts | Dispatched by built-in `delegate-task` tool, Agents have independent model/fallback/tools config |
| **Persistence** | Stateless (context rebuilt each dispatch) | Stateful (session_id maintains continuity) |

**Superpowers has 4 types of "roles", but only 1 is formally named as an Agent.** The other 3 are sub-roles defined through Prompt Templates in the `subagent-driven-development` Skill. oh-my-opencode implements all roles as formal Agents.

---

## 1. Complete Agent Role Inventory

### Superpowers' 4 Role Types

| # | Role | Source | Definition Method | Responsibilities |
|---|------|--------|-------------------|-----------------|
| SP-1 | **Controller** | Implicit in `subagent-driven-development` Skill | Skill SKILL.md describes process | Read plan → extract tasks → create Todo → dispatch sub-agents → review results → advance to next |
| SP-2 | **Implementer** | `implementer-prompt.md` | Prompt Template | Receive specific task → ask clarifying questions → implement code → write tests → self-review → report status |
| SP-3 | **Spec Reviewer** | `spec-reviewer-prompt.md` | Prompt Template | Read actual code → compare against spec line by line → check for omissions/extras/misunderstandings → report compliance |
| SP-4 | **Code Reviewer** | `agents/code-reviewer.md` + `code-quality-reviewer-prompt.md` | Formal Agent + Prompt Template | Review code quality: architecture, SOLID, separation of concerns, test coverage, documentation |

### oh-my-opencode's 11 Agents

| # | Agent | Model | Responsibilities |
|---|-------|-------|-----------------|
| OMO-1 | **Sisyphus** | claude-opus-4-7 max | Main orchestrator: planning, delegation, verification, parallel execution |
| OMO-2 | **Hephaestus** | gpt-5.4 medium | Autonomous deep worker: given goals not steps, end-to-end execution |
| OMO-3 | **Oracle** | gpt-5.4 high | Read-only consultant: architecture decisions, complex debugging, consult after 2+ failed fixes |
| OMO-4 | **Librarian** | minimax-m2.7 | External search: docs, open-source code, best practices |
| OMO-5 | **Explore** | grok-code-fast-1 | Codebase Grep: structure, patterns, file location |
| OMO-6 | **Prometheus** | claude-opus-4-7 max | Strategic planning: interviewer mode to collect requirements → build detailed plan |
| OMO-7 | **Metis** | claude-opus-4-7 max (temp 0.3) | Pre-planning consultant: intent classification, ambiguity detection, AI failure prevention |
| OMO-8 | **Momus** | gpt-5.4 xhigh | Plan reviewer: verify plan executability, reference validity |
| OMO-9 | **Atlas** | claude-sonnet-4-6 | Todo orchestrator: progress tracking, task continuation |
| OMO-10 | **Multimodal-Looker** | gpt-5.3-codex | Multimodal analysis: PDF, images, charts |
| OMO-11 | **Sisyphus-Junior** | claude-sonnet-4-6 | Category executor: derived by Sisyphus based on category |

---

## 2. Mapping Relationships

### 2.1 Core Mapping Table

```
Superpowers                          oh-my-opencode
─────────────                        ──────────────

[Controller role]          ←────→    Sisyphus (Main Orchestrator)
 subagent-driven-development
 "you" (main Agent)                   Similarities: Read plan → dispatch sub-agents → review → advance
                                       Differences: Sisyphus has IntentGate, dynamic Prompt,
                                             multi-model fallback, parallel background Agents

[Implementer role]         ←────→    Hephaestus / Sisyphus-Junior
 implementer-prompt.md                Similarities: Receive task → implement → test → report
                                       Differences: Hephaestus is "given goals not steps" autonomous mode
                                             Junior is a lightweight executor derived by category
                                             Superpowers' Implementer is more "execute by the book"

[Spec Reviewer role]       ←────→    Momus (Plan Reviewer)
 spec-reviewer-prompt.md              Similarities: Verify implementation matches spec/plan
                                       Differences: Momus focuses on "plan executability" not "implementation compliance"
                                             Momus reviews plan files, Spec Reviewer reviews code

[Code Reviewer Agent]      ←────→    Momus (Quality Review) + Oracle (Architecture Review)
 agents/code-reviewer.md              Similarities: Code quality, architecture, test coverage
                                       Differences: OmO splits review into two specialized Agents
                                             Momus focuses on plan/executability
                                             Oracle focuses on architecture/deep reasoning

[brainstorming Skill]      ←────→    Prometheus (Interviewer Mode Planning)
 implicit "designer" role             Similarities: Ask questions → refine requirements → design → get approval
                                       Differences: Prometheus has Metis pre-analysis
                                             Superpowers has main Agent directly execute brainstorming
                                             OmO splits planning into Prometheus + Metis two phases

(writing-plans Skill)      ←────→    Prometheus (Plan Generation)
 implicit "planner" role              Similarities: Break design into atomic tasks
                                       Differences: Prometheus uses YAML format with parallel task graph
                                             Superpowers uses Markdown checkbox format

(No equivalent)            ────→    Metis (Pre-planning Consultant)
                                       OmO unique: Analyzes intent and detects ambiguities before planning
                                       Superpowers embeds this responsibility in brainstorming Skill

(No equivalent)            ────→    Explore (Codebase Grep)
                                       OmO unique: Specialized fast code search Agent
                                       Superpowers depends on host platform's search tools

(No equivalent)            ────→    Librarian (External Search)
                                       OmO unique: External docs/open-source code search Agent
                                       Superpowers doesn't provide this capability

(No equivalent)            ────→    Atlas (Todo Orchestrator)
                                       OmO unique: Dedicated Todo progress tracking and task continuation
                                       Superpowers depends on main Agent manually managing Todo

(No equivalent)            ────→    Multimodal-Looker
                                       OmO unique: PDF/image/chart analysis
```

### 2.2 Role Mapping Visualization

```
                    Superpowers                              oh-my-opencode
                    ───────────                              ──────────────

               ┌──────────────────┐                    ┌──────────────────┐
               │   Controller     │                    │    Sisyphus      │
               │ (Main/Orchestrate)│◄──────maps──────► │   (Orchestrator) │
               └──────────────────┘                    └──────────────────┘
                       │ dispatch                              │ dispatch
          ┌────────────┼────────────┐              ┌──────────┼──────────┐
          ▼            ▼            ▼              ▼          ▼          ▼
   ┌──────────┐ ┌──────────┐ ┌──────────┐  ┌──────────┐ ┌────────┐ ┌──────────┐
   │Implementer│ │  Spec    │ │  Code    │  │Hephaestus│ │ Momus  │ │ Oracle   │
   │ (Worker)  │ │ Reviewer │ │ Reviewer │  │(Autonomous)│ │(Plan)  │ │(Advisor) │
   └──────────┘ └──────────┘ └──────────┘  └──────────┘ └────────┘ └──────────┘
                                                       ┌────────┐ ┌────────┐
                                                       │ Junior │ │ Atlas  │
                                                       │(Category)│ │(Todo)  │
                                                       └────────┘ └────────┘

               ┌──────────────────┐                    ┌──────────────────┐
               │  brainstorming   │                    │   Prometheus     │
               │  (Design/Plan)   │◄──────maps──────► │  (Strategic Plan)│
               └──────────────────┘                    └──────────────────┘
                                                              │ pre-step
                                                      ┌────────┐
                                                      │ Metis  │ ◄── OmO unique
                                                      │(Intent) │
                                                      └────────┘

                                                      ┌────────┐ ┌──────────┐
                                                      │Explore │ │Librarian │ ◄── OmO unique
                                                      │(Code)  │ │(External)│
                                                      └────────┘ └──────────┘
```

---

## 3. Role-by-Role Deep Comparison

### 3.1 Main Orchestrator Role

| Dimension | SP: Controller (implicit) | OMO: Sisyphus |
|-----------|--------------------------|---------------|
| **Definition Method** | "You" in Skill text | TypeScript factory function + dynamic Prompt |
| **Model** | Inherits from host platform | claude-opus-4-7 max (5-level fallback) |
| **Delegation Method** | Via host Task tool, Prompt Template guidance | Built-in `delegate-task` tool, 8 category routes |
| **Parallel Capability** | Depends on host platform | Native background Agents (5 concurrent/model, configurable) |
| **Context Management** | Manually extracts task text for sub-Agents | Dynamically builds Prompt, session_id maintains continuity |
| **Intent Recognition** | No dedicated mechanism | IntentGate (6 intent categories) |
| **Review Process** | Two-stage: Spec → Code Quality | On-demand: Oracle/Momus/self-review |
| **Plan Awareness** | Reads all tasks at once | Continuously tracks progress via Atlas |

**Key Difference**: Sisyphus is a "Controller with infrastructure support", possessing model routing, background parallelism, session continuity and other capabilities that Superpowers Controller lacks. However, Superpowers Controller has stricter process discipline (two-stage review is mandatory).

### 3.2 Implementation/Execution Role

| Dimension | SP: Implementer | OMO: Hephaestus | OMO: Sisyphus-Junior |
|-----------|----------------|-----------------|---------------------|
| **Definition Method** | Prompt Template | TypeScript factory | TypeScript factory |
| **Autonomy Level** | Low (execute by the book) | High (given goals not steps) | Medium (execute by category) |
| **Question-asking Ability** | Yes (both before starting and during work) | Yes | Yes |
| **Self-review Requirement** | Mandatory (5-item checklist) | No explicit requirement | No explicit requirement |
| **Status Report** | 4 states: DONE/CONCERNS/BLOCKED/NEEDS_CONTEXT | No standard format | No standard format |
| **Model Strategy** | Select model by task complexity | Fixed gpt-5.4 | Route by category |
| **Applicable Scenarios** | Single task with clear spec | Complex end-to-end work | Standard tasks dispatched by category |

**Key Difference**: Superpowers' Implementer is a "by-the-book engineer", emphasizing strict spec compliance and self-review; Hephaestus is an "independent craftsman", given goals and exploring execution paths autonomously. This is a philosophical difference of **discipline vs. autonomy**.

### 3.3 Review Roles

| Dimension | SP: Spec Reviewer | SP: Code Reviewer | OMO: Momus | OMO: Oracle |
|-----------|-------------------|-------------------|------------|-------------|
| **Review Target** | Implementation code vs. spec | Code quality | Plan files | Any technical issue |
| **Review Timing** | After implementation | After Spec review passes | After plan generation | Anytime |
| **Core Question** | "Did it implement what the spec requires?" | "Is the code well-written?" | "Can the plan be executed?" | "What's the right approach?" |
| **Output Format** | ✅/❌ + issue list | Strengths/Issues/Assessment | [OKAY]/[REJECT] + blocking issues | Recommended approach |
| **Strictness** | Strict line-by-line comparison | Standard Code Review | Leans toward approval (80% sufficient) | As deep as needed |
| **Write Permission** | None (read-only) | None (read-only) | None (read-only) | None (read-only) |

**Key Difference**: Superpowers' review is an **embedded mandatory step** in the process (two-stage review in subagent-driven-development workflow), while Momus/Oracle are **on-demand consultants**. Superpowers reviews spec compliance first then code quality; OmO doesn't have this strict two-stage separation.

### 3.4 Planning Roles

| Dimension | SP: brainstorming Skill | OMO: Prometheus | OMO: Metis |
|-----------|------------------------|-----------------|------------|
| **Trigger Method** | Automatic (Skill detects creative work) | User `/start-work` or Sisyphus delegation | Sisyphus detects complex task |
| **Input** | User's raw idea | Metis analysis + user conversation | User request |
| **Output** | Design document → Implementation plan | YAML plan file (with parallel task graph) | Intent analysis + questions + risks + Prometheus instructions |
| **Interaction Mode** | Q&A, gradual refinement | Interviewer mode + codebase exploration | Analysis mode (can call explore/librarian) |
| **Depth** | Includes Visual Companion | Includes high-precision mode | Includes 6 intent classification strategies |
| **Prerequisites** | None | Metis pre-analysis | None |

**Key Difference**: OmO splits planning into a **Metis (pre-analysis) → Prometheus (planning) → Momus (review)** three-stage pipeline; Superpowers compresses planning into **brainstorming → writing-plans** two steps, executed by the same main Agent.

---

## 4. OmO-Exclusive Roles (No SP Equivalent)

| Agent | Responsibility | Why SP Doesn't Need It |
|-------|---------------|----------------------|
| **Explore** | Fast codebase search | SP doesn't provide tool layer, depends on host platform |
| **Librarian** | External docs/code search | SP doesn't provide external search capability |
| **Atlas** | Todo progress tracking and continuation | SP depends on main Agent manually managing Todo |
| **Multimodal-Looker** | PDF/image/chart analysis | SP doesn't handle multimodal |
| **Metis** | Pre-planning intent analysis | SP embeds this in brainstorming Skill |

---

## 5. SP-Exclusive Capabilities (No OmO Equivalent)

| Capability | Source | Why OmO Doesn't Have It |
|-----------|--------|------------------------|
| **Mandatory Two-Stage Review** | subagent-driven-development | OmO's review is on-demand, not process-mandatory |
| **Implementer Self-Review Checklist** | implementer-prompt.md | OmO's Agents don't have embedded self-review processes |
| **4 Status Report Types** | implementer-prompt.md | OmO doesn't have a standardized Agent status protocol |
| **receiving-code-review Skill** | Complete feedback reception process | OmO doesn't have corresponding "how to respond to reviews" guidance |
| **TDD Iron Law** | test-driven-development Skill | OmO doesn't enforce TDD at the Agent level |
| **4-Stage Systematic Debugging Process** | systematic-debugging Skill | OmO doesn't enforce debugging methodology at the Agent level |

---

## 6. Common Patterns Summary

Despite vastly different implementations, the two frameworks share several deep patterns in Agent role design:

### 6.1 Controller-Worker Separation
Both separate "orchestration" and "execution" into different roles. SP's Controller dispatches Implementers, Sisyphus dispatches Hephaestus/Junior.

### 6.2 Review-Implementation Separation
Both have separation between "doers" and "checkers". SP uses Spec Reviewer + Code Reviewer, OmO uses Momus + Oracle.

### 6.3 Planning-Execution Separation
Both separate "thinking clearly" from "doing work". SP uses brainstorming → writing-plans → execution, OmO uses Metis → Prometheus → Momus → execution.

### 6.4 Read-Only Consultant Pattern
Both have consultant roles (Oracle, Code Reviewer) that are read-only — they can analyze and advise but not modify files.

### 6.5 Context Isolation
Both emphasize that sub-Agents don't inherit main session context. SP's Prompt Template explicitly requires "never inherit your session's context", OmO achieves natural isolation through Background Agents.

---

## 7. Mapping Summary Table

| Functional Area | Superpowers Role | oh-my-opencode Role | Mapping Relationship |
|----------------|------------------|--------------------|--------------------|
| **Main Orchestration** | Controller (implicit) | Sisyphus | **Direct mapping** |
| **Task Execution** | Implementer (Template) | Hephaestus + Junior | **One-to-many**, Hephaestus more autonomous |
| **Spec Review** | Spec Reviewer (Template) | Momus | **Partial mapping**, different review targets |
| **Code Review** | Code Reviewer (Agent) | Momus + Oracle | **One-to-many**, OmO splits into two specialists |
| **Requirements Analysis** | brainstorming Skill | Prometheus + Metis | **One-to-many**, OmO splits into pre-analysis + planning |
| **Plan Writing** | writing-plans Skill | Prometheus | **Direct mapping** |
| **Plan Review** | None (writing-plans includes self-review) | Momus | **OmO-unique** independent plan review role |
| **Code Search** | None | Explore | **OmO-unique** |
| **External Search** | None | Librarian | **OmO-unique** |
| **Progress Tracking** | None | Atlas | **OmO-unique** |
| **Multimodal** | None | Multimodal-Looker | **OmO-unique** |
| **Intent Pre-analysis** | None | Metis | **OmO-unique** |
| **TDD Discipline** | TDD Skill (full-process mandatory) | No equivalent Agent | **SP-unique** |
| **Debugging Methodology** | Systematic Debugging Skill | No equivalent Agent | **SP-unique** |
| **Feedback Reception** | receiving-code-review Skill | No equivalent | **SP-unique** |

**One-line conclusion**: Superpowers completes the development cycle with **4 roles × strict process discipline**; oh-my-opencode completes the same cycle with **11 Agents × specialized division of labor**. Superpowers' roles are "different processes of the same type of worker", oh-my-opencode's roles are "different trades of specialized professionals".
