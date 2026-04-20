# Oh My Superpowers

Superpowers' workflow discipline meets oh-my-opencode's agent orchestration.

This is a fork of [obra/superpowers](https://github.com/obra/superpowers) that integrates with [oh-my-opencode](https://github.com/code-yeongyu/oh-my-opencode) — combining Superpowers' strict development process with oh-my-opencode's 11 specialized agents, multi-model routing, and tool infrastructure.

## Why This Exists

| Layer | Superpowers (original) | oh-my-opencode |
|-------|----------------------|----------------|
| **What it provides** | Workflow discipline (brainstorm → plan → TDD → review) | Agent orchestration (11 agents, multi-model, tools) |
| **Constraint method** | Prompt-based iron laws | Hook-based interception + tool precision |
| **Trust model** | Doesn't trust AI self-discipline | Doesn't trust any single model |
| **Philosophy** | "Discipline over inspiration" | "Orchestrate everything, don't lock in" |

**Together**: Superpowers defines *how work should be done*, oh-my-opencode provides *what to use and how to schedule it*.

```
┌──────────────────────────────────────────────┐
│   Oh My Superpowers                          │
│                                              │
│   Superpowers Skills (discipline layer)      │
│   - TDD, debugging, review processes         │
│   - Brainstorming, plan writing methodology  │
│   - Two-stage review (spec → quality)        │
│                                              │
│   oh-my-opencode Agents (orchestration)      │
│   - Sisyphus (main), Oracle (review)         │
│   - Prometheus (planning), Metis (analysis)  │
│   - Momus (spec review), Explore, Librarian   │
│   - Multi-model routing, session continuity  │
└──────────────────────────────────────────────┘
```

## Agent Role Mapping

The core integration: each Superpowers workflow role maps to an oh-my-opencode agent.

| Superpowers Role | oh-my-opencode Agent | Dispatch |
|---|---|---|
| **Controller** (orchestrator) | **Sisyphus** | Main agent, no dispatch needed |
| **Implementer** (simple) | **Sisyphus-Junior** via `category="quick"` | Fast model, TDD skill loaded |
| **Implementer** (complex) | **Sisyphus-Junior** via `category="deep"` | Autonomous model, explores codebase |
| **Implementer** (hard logic) | **Sisyphus-Junior** via `category="ultrabrain"` | High-reasoning model |
| **Implementer** (frontend) | **Sisyphus-Junior** via `category="visual-engineering"` | Creative model |
| **Spec Reviewer** | **Momus** | `subagent_type="momus"` — verifies implementation matches spec |
| **Code Quality Reviewer** | **Oracle** | `subagent_type="oracle"` — architecture, quality, deep review |
| **Requirements Analysis** | **Prometheus + Metis** | Metis pre-analyzes → Prometheus plans |
| **Codebase Search** | **Explore** | `subagent_type="explore"`, background |
| **External Research** | **Librarian** | `subagent_type="librarian"`, background |

## The Workflow

```
1. brainstorming     → Sisyphus asks what you really want, designs with you
2. using-git-worktrees → Isolated workspace on new branch
3. writing-plans     → Prometheus builds detailed, atomic task plan
4. subagent-driven-development → Per-task dispatch with two-stage review:
   ├── Implementer (via task category) → writes code + tests
   ├── Spec Reviewer (Momus)           → "did it build what was asked?"
   └── Code Reviewer (Oracle)          → "is it well-built?"
5. test-driven-development → RED-GREEN-REFACTOR enforced
6. finishing-a-development-branch → merge/PR/cleanup
```

### Two-Stage Review (the key discipline)

Every implementation task goes through two independent reviews:

1. **Spec compliance** (Momus) — "Did it implement what the spec requires? Nothing more, nothing less."
2. **Code quality** (Oracle) — "Is it well-architected, tested, maintainable?"

Issues found in either review go back to the implementer (via `session_id` continuity), then re-reviewed. Neither stage is skippable.

## Skills

All skills are prefixed `omo-` to indicate oh-my-opencode integration. They use OmO's `task()` API for sub-agent dispatch.

### Development Flow

| Skill | Purpose |
|-------|---------|
| `omo-brainstorming` | Socratic design refinement before code |
| `omo-writing-plans` | Atomic implementation plans |
| `omo-using-git-worktrees` | Isolated workspace setup |
| `omo-subagent-driven-development` | Per-task dispatch with two-stage review |
| `omo-executing-plans` | Batch execution with human checkpoints |
| `omo-dispatching-parallel-agents` | Concurrent sub-agent workflows |
| `omo-test-driven-development` | RED-GREEN-REFACTOR enforcement |
| `omo-verification-before-completion` | Evidence before assertions |
| `omo-systematic-debugging` | 4-phase root cause analysis |

### Review & Completion

| Skill | Purpose |
|-------|---------|
| `omo-requesting-code-review` | Pre-review checklist |
| `omo-receiving-code-review` | Responding to feedback with rigor |
| `omo-finishing-a-development-branch` | Merge/PR decision workflow |

### Meta

| Skill | Purpose |
|-------|---------|
| `omo-using-superpowers` | Skill discovery and auto-injection |
| `omo-writing-skills` | Create new skills |
| `omo-sp-omo-bridge` | SP role → OmO agent routing table |

## Bridge Skill

The `omo-sp-omo-bridge` skill provides a declarative mapping table that Sisyphus auto-reads when dispatching sub-agents. It maps:

- **Role → agent type**: Which OmO agent handles which SP role
- **Skill injection**: Which skills to auto-load per task type
- **Two-stage review**: Spec (Momus) → Quality (Oracle) enforcement
- **Session continuity**: How to use `session_id` for follow-up dispatches

This is the integration described in [inspire/05-integration-guide.md](https://github.com/JerryDeng0425/oh-my-superpowers/blob/main/inspire/05-integration-guide.md), Section C — minimal intrusion, adjustable anytime.

## Architecture

```
oh-my-superpowers/
├── skills/                      # 15 OmO-integrated skills (omo- prefix)
│   ├── omo-sp-omo-bridge/       # Bridge: SP roles → OmO agents
│   ├── omo-subagent-driven-development/
│   │   ├── SKILL.md             # Controller process with OmO task() API
│   │   ├── implementer-prompt.md
│   │   ├── spec-reviewer-prompt.md
│   │   └── code-quality-reviewer-prompt.md
│   └── ...                      # Other workflow skills
├── agents/
│   └── code-reviewer.md         # Reference prompt (dispatched via Oracle)
├── inspire/                     # Analysis documents behind this integration
│   ├── 01-superpowers-analysis.md
│   ├── 02-oh-my-opencode-analysis.md
│   ├── 03-comparison.md
│   ├── 04-agent-role-mapping.md
│   ├── 05-integration-guide.md
│   └── 06-opencode-native-install-analysis.md
├── hooks/                       # Session start hooks
├── commands/                    # Slash commands
└── tests/                       # Integration tests
```

## Inspiration

The `inspire/` directory contains the analysis work that led to this integration:

1. **[Superpowers analysis](https://github.com/JerryDeng0425/oh-my-superpowers/blob/main/inspire/01-superpowers-analysis.md)** — 14 skills, 1 agent, pure Markdown workflow framework
2. **[oh-my-opencode analysis](https://github.com/JerryDeng0425/oh-my-superpowers/blob/main/inspire/02-oh-my-opencode-analysis.md)** — 11 agents, 52 hooks, full TypeScript orchestration system
3. **[Comparison](https://github.com/JerryDeng0425/oh-my-superpowers/blob/main/inspire/03-comparison.md)** — "Superpowers is the discipline manual; OmO is the operating system"
4. **[Agent role mapping](https://github.com/JerryDeng0425/oh-my-superpowers/blob/main/inspire/04-agent-role-mapping.md)** — SP's 4 roles mapped to OmO's 11 agents
5. **[Integration guide](https://github.com/JerryDeng0425/oh-my-superpowers/blob/main/inspire/05-integration-guide.md)** — Three integration paths (A: modify skills, B: custom categories, C: bridge skill)
6. **[OpenCode native install analysis](https://github.com/JerryDeng0425/oh-my-superpowers/blob/main/inspire/06-opencode-native-install-analysis.md)** — Installation strategy

Also see the original [README-ORIGIN.md](https://github.com/JerryDeng0425/oh-my-superpowers/blob/main/README-ORIGIN.md) from the upstream Superpowers project.

**One-line conclusion**: Superpowers completes the development cycle with **4 roles × strict process discipline**; oh-my-opencode completes the same cycle with **11 agents × specialized division of labor**. Oh My Superpowers combines both.

## Installation

### OpenCode (via npm)

This plugin is designed for OpenCode with oh-my-opencode installed.

Add to your `opencode.json` (global `~/.config/opencode/opencode.json` or project-level):

```json
{
  "plugin": ["oh-my-superpowers"]
}
```

Or install via CLI:

```bash
opencode plugin install oh-my-superpowers
```

Pin a specific version:

```json
{
  "plugin": ["oh-my-superpowers@^5.0.7"]
}
```

### Prerequisites

- [OpenCode](https://opencode.ai) installed
- [oh-my-opencode](https://github.com/code-yeongyu/oh-my-opencode) plugin installed

```bash
# Install oh-my-opencode first if not already
opencode plugin install oh-my-opencode
```

### Verify

Start a new OpenCode session and ask: "Tell me about your superpowers". The agent should recognize and list the available skills.

### Publishing (maintainers)

```bash
# Bump version
./scripts/bump-version.sh <new-version>

# Dry run — verify what gets packed
npm pack --dry-run

# Publish to npm
npm publish
```

## Philosophy

- **Test-Driven Development** — Write tests first, always
- **Two-stage review** — Spec compliance, then code quality, neither skippable
- **Systematic over ad-hoc** — Process over guessing
- **Evidence over claims** — Verify before declaring success
- **Right model for the job** — Fast model for simple tasks, reasoning model for hard ones
- **Discipline + orchestration** — Superpowers says *how*, OmO provides *what*

## License

MIT License — see LICENSE file for details.

Based on [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent (Prime Radiant), integrated with [oh-my-opencode](https://github.com/code-yeongyu/oh-my-opencode) by YeonGyu Kim.
