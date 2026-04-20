# Superpowers vs oh-my-opencode Comparison Analysis Report

> **Analysis Date**: 2026-04-19  
> **Versions**: superpowers v5.0.7 / oh-my-opencode v3.17.4

---

## 1. One-Line Summary

**Superpowers is the workflow discipline manual for AI coding; oh-my-opencode is the complete operating system for AI coding.**

They solve different layers of the same domain (AI-assisted software development) and can be used complementarily.

---

## 2. Positioning Comparison

| Dimension | Superpowers | oh-my-opencode |
|-----------|-------------|----------------|
| **Essence** | Prompt-as-Code workflow framework | Full-featured Agent orchestration plugin |
| **Abstraction Layer** | Workflow/discipline layer | Tool/Agent/infrastructure layer |
| **Dependencies** | Only requires AI to read Markdown | Requires OpenCode platform + Bun Runtime |
| **Target Users** | All AI coding users | OpenCode users |
| **Core Philosophy** | "Discipline over inspiration" | "Orchestrate all models" |
| **Code Volume** | ~14 Markdown files | 1,766 TypeScript files / 377k LOC |
| **License** | MIT | SUL-1.0 |

---

## 3. Architecture Comparison

### 3.1 Architecture Layers

```
┌──────────────────────────────────────────────┐
│            User / Developer                    │
├──────────────────────────────────────────────┤
│   Superpowers (Workflow Discipline Layer)    │  ← Pure Markdown
│   - Skills: brainstorming, TDD, debugging    │
│   - Defines "how things should be done"      │
├──────────────────────────────────────────────┤
│   oh-my-opencode (Agent Orchestration Layer) │  ← TypeScript
│   - 11 Agents, 26 Tools, 52 Hooks            │
│   - Defines "what to use, how to schedule"   │
├──────────────────────────────────────────────┤
│   OpenCode (Platform Layer)                   │  ← Go / External
│   - LSP Server, File System, Process Mgmt    │
│   - Provides "infrastructure"                │
└──────────────────────────────────────────────┘
```

Superpowers runs at the top layer, constraining Agent behavior through Prompts; oh-my-opencode runs at the middle layer, providing Agent, tool, and Hook infrastructure.

### 3.2 Extension Model

| Aspect | Superpowers | oh-my-opencode |
|--------|-------------|----------------|
| **Extension Unit** | Skill (SKILL.md) | Agent / Tool / Hook / Skill / Command / MCP |
| **Extension Language** | Markdown | TypeScript |
| **Extension Difficulty** | Very low (write Markdown) | Medium (requires understanding plugin architecture) |
| **Extension Discovery** | `using-superpowers` auto-injection | `categorySkillReminder` Hook + Skill Loader |
| **Runtime Registration** | None (AI reads directly) | `createTools()` / `createHooks()` factory registration |

---

## 4. Feature Comparison

### 4.1 Workflow Coverage

| Development Phase | Superpowers | oh-my-opencode |
|-------------------|-------------|----------------|
| **Requirements Analysis** | `brainstorming` Skill | Prometheus Agent (`/start-work`) |
| **Design** | `brainstorming` → Design document | Prometheus interviewer mode |
| **Planning** | `writing-plans` Skill | Prometheus → Implementation plan |
| **Execution** | `subagent-driven-development` / `executing-plans` | `delegate-task` Tool + Background Agents |
| **Testing** | `test-driven-development` Skill | No dedicated TDD Skill (depends on Skill loader) |
| **Debugging** | `systematic-debugging` Skill | No dedicated debugging Skill |
| **Code Review** | `requesting-code-review` + `code-reviewer` Agent | Momus Agent |
| **Completion** | `finishing-a-development-branch` | Atlas Agent + Todo Enforcer |
| **Verification** | `verification-before-completion` Skill | No dedicated verification Skill |

**Key Difference**: Superpowers has more complete and stricter workflow coverage; oh-my-opencode is more powerful at the Agent orchestration and tool layer.

### 4.2 Agent System

| Aspect | Superpowers | oh-my-opencode |
|--------|-------------|----------------|
| **Agent Count** | 1 (code-reviewer) | 11 specialized Agents |
| **Agent Implementation** | Markdown Prompt definition | TypeScript factory functions + dynamic Prompts |
| **Multi-model Support** | Inherits from host platform | Native multi-model routing |
| **Fallback Chain** | None | Each Agent independently configured |
| **Context Isolation** | Depends on host platform | Background Agent natural isolation |
| **Model Selection** | Not applicable | 4-step resolution + category routing |

### 4.3 Tool System

| Tool Category | Superpowers | oh-my-opencode |
|---------------|-------------|----------------|
| **LSP** | None | `lsp_rename`, `lsp_goto_definition`, `lsp_find_references`, `lsp_diagnostics`, `lsp_symbols` |
| **AST-Grep** | None | `ast_grep_search`, `ast_grep_replace` |
| **File Editing** | None (depends on host) | Hashline Edit (LINE#ID hash-anchored) |
| **Terminal** | None | Interactive Bash (Tmux) |
| **Session Management** | None | Session List/Read/Search/Info |
| **Multimodal** | None | Look At (PDF/image/chart analysis) |
| **Web Search** | None | Built-in Exa + Tavily |
| **Documentation Lookup** | None | Built-in Context7 |
| **GitHub Search** | None | Built-in Grep.app |
| **Sub-Agents** | None | Task Tool (with category routing) |

**Conclusion**: oh-my-opencode provides a complete tool stack; Superpowers provides no tools, fully depending on the host platform.

### 4.4 Hook System

| Aspect | Superpowers | oh-my-opencode |
|--------|-------------|----------------|
| **Hook Count** | 1 (SessionStart) | 52 |
| **Hook Layers** | Flat | 3 layers (Session → ToolGuard → Transform → Continuation → Skill) |
| **Intent Classification** | None | IntentGate |
| **Context Monitoring** | None | contextWindowMonitor |
| **Model Fallback** | None | modelFallback + runtimeFallback |
| **Comment Detection** | None | commentChecker |
| **Compaction Preservation** | None | compactionContextInjector + compactionTodoPreserver |

---

## 5. Design Philosophy Comparison

### 5.1 Superpowers' Philosophy

> **"Discipline over inspiration"**

- Trusts AI's capability, doesn't trust AI's discipline
- Enforces processes through Iron Laws, Hard Gates, Red Flags
- "Even if there's only a 1% chance of relevance, must invoke Skill"
- TDD is not a suggestion, it's an iron law — "Write code before the test? Delete it. Start over."
- Every decision needs evidence support

### 5.2 oh-my-opencode's Philosophy

> **"Orchestrate everything, don't lock in"**

- Doesn't trust a single model, trusts specialized division of labor
- Each model does what it's best at (Claude for planning, GPT for reasoning, Minimax for speed, Gemini for creativity)
- Tools matter more than Prompts — "Most Agent failures aren't model problems, they're editing tool problems"
- Enforces discipline through Hook system rather than Prompts
- "If OpenCode is Debian, OmO is Ubuntu"

### 5.3 Philosophy Differences Summary

| Dimension | Superpowers | oh-my-opencode |
|-----------|-------------|----------------|
| **Constraint Method** | Prompt constraints (discipline) | Hook interception (mechanism) |
| **Trust Model** | Doesn't trust Agent self-discipline | Doesn't trust single model capability |
| **Reliability Source** | Workflow discipline | Tool precision |
| **Complexity Strategy** | Minimalist (Markdown only) | Full-featured (TypeScript full-stack) |
| **Scope** | All AI platforms | OpenCode ecosystem |

---

## 6. Complementarity Analysis

### 6.1 Can They Be Used Together?

**Yes, and they're naturally compatible.**

oh-my-opencode's Skill loader (`src/features/opencode-skill-loader/`) supports loading standard SKILL.md format skills. This means Superpowers' 14 Skills can be loaded and used as oh-my-opencode skills.

In fact, oh-my-opencode's Sisyphus Agent already has built-in orchestration logic similar to Superpowers (delegation, review, TDD), but Superpowers provides more detailed process guidance and discipline constraints.

### 6.2 Complementarity Matrix

| Scenario | Recommended Approach |
|----------|---------------------|
| Only using Claude Code, need workflow discipline | Superpowers |
| Only using OpenCode, need Agent orchestration | oh-my-opencode |
| OpenCode + need strict workflow | oh-my-opencode + Superpowers Skills |
| Multi-platform users | Superpowers (cross-platform) + platform-specific tools |
| Need precise tools like LSP/AST | oh-my-opencode |
| Need multi-model orchestration | oh-my-opencode |
| Need TDD/debugging process discipline | Superpowers |

### 6.3 Complementary Design

```
oh-my-opencode provides:              Superpowers provides:
┌──────────────────┐              ┌──────────────────┐
│  Agent Orchestration │              │  Workflow Discipline │
│  Tool Infrastructure │  ──complementary──▶   │  TDD Process         │
│  Hook Interception   │              │  Debugging Methodology│
│  Multi-model Routing │              │  Design Review Flow  │
│  MCP Management      │              │  Verification Checklist│
│  Session Management  │              │  Plan Writing Method │
└──────────────────┘              └──────────────────┘
        ▲                                  ▲
        │         Skill Interface          │
        └──────── SKILL.md ────────────────┘
```

---

## 7. Summary

### Superpowers

- **Positioning**: AI coding discipline framework
- **Strengths**: Extremely lightweight, cross-platform, complete workflow, strong discipline
- **Weaknesses**: No tool layer, no Agent orchestration, no state management
- **Best for**: Developers pursuing engineering discipline, using multiple platforms

### oh-my-opencode

- **Positioning**: Full-featured AI Agent orchestration system
- **Strengths**: Rich Agents, complete tools, deep Hooks, multi-model orchestration
- **Weaknesses**: High complexity, platform-bound, non-standard license
- **Best for**: OpenCode users, developers needing multi-model orchestration

### Relationship

The two are **complementary projects at different abstraction layers** within the same ecosystem. Superpowers solves the "**how things should be done**" problem, oh-my-opencode solves the "**what to use and how to schedule**" problem. They are naturally compatible through the SKILL.md interface and can be layered for the best experience.
