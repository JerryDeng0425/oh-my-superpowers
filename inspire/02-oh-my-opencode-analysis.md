# oh-my-opencode Project Architecture Analysis Report

> **Project**: code-yeongyu/oh-my-opencode v3.17.4  
> **Positioning**: Full-featured AI Agent orchestration framework — "The Best AI Agent Harness"  
> **Author**: YeonGyu Kim (code-yeongyu)  
> **License**: SUL-1.0  
> **Analysis Date**: 2026-04-19

---

## 1. Project Overview

oh-my-opencode (abbreviated OmO) is a **full-featured plugin** for the OpenCode platform, providing 11 AI Agents, 52 lifecycle Hooks, 26 tools, a 3-tier MCP system, and a complete Claude Code compatibility layer. It is one of the most comprehensive AI coding Agent enhancement frameworks available today.

Core philosophy: **Don't lock into models, orchestrate everything**. It simultaneously uses Claude, Kimi, GPT, Gemini, GLM, Minimax and other models, letting each model do what it does best.

### Project Scale

| Metric | Value |
|--------|-------|
| TypeScript Source Files | 1,766 |
| Lines of Code | 377,000+ |
| Barrel Exports (index.ts) | 104 |
| Agent Definitions | 11 |
| Lifecycle Hooks | 52 |
| Tools | 26 |
| Feature Modules | 19 |
| Built-in MCPs | 3 |
| Config Files | 32 |
| Platform Binaries | 11 |

---

## 2. Core Architecture

### 2.1 Overall Architecture Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                      OpenCode Platform                        │
│                   (@opencode-ai/plugin)                       │
└─────────────────────┬───────────────────────────────────────┘
                      │ Plugin Interface (10 Hook Handlers)
                      ▼
┌──────────────────────────────────────────────────────────────┐
│                    oh-my-opencode Plugin                       │
│                                                                │
│   ┌─────────────────── Initialization ───────────────────┐    │
│   │  loadConfig → createManagers → createTools           │    │
│   │            → createHooks → createPluginInterface     │    │
│   └──────────────────────────────────────────────────────┘    │
│                                                                │
│   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────┐     │
│   │ 11 Agents │ │ 26 Tools │ │ 52 Hooks │ │ 3 MCPs     │     │
│   │           │ │          │ │          │ │ (built-in) │     │
│   │ Sisyphus  │ │ LSP      │ │ Session  │ │ WebSearch  │     │
│   │ Hephaestus│ │ AST-Grep │ │ ToolGuard│ │ Context7   │     │
│   │ Oracle    │ │ Hashline │ │ Transform│ │ Grep.app   │     │
│   │ Librarian │ │ Session  │ │ Continue │ │            │     │
│   │ Explore   │ │ Tmux     │ │ Skill    │ │            │     │
│   │ Prometheus│ │ Task     │ │          │ │            │     │
│   │ Metis     │ │ Glob     │ │          │ │            │     │
│   │ Momus     │ │ Grep     │ │          │ │            │     │
│   │ Atlas     │ │ LookAt   │ │          │ │            │     │
│   │ Multimodal│ │ SlashCmd │ │          │ │            │     │
│   │ Junior    │ │ ...      │ │          │ │            │     │
│   └──────────┘ └──────────┘ └──────────┘ └────────────┘     │
│                                                                │
│   ┌────────────────┐ ┌──────────────┐ ┌────────────────┐     │
│   │ 19 Features    │ │ Config (Zod) │ │ CLI (Commander)│     │
│   │                │ │ 32 files     │ │ install/doctor │     │
│   │ BackgroundAgent│ │ JSONC multi- │ │ run/mcp-oauth  │     │
│   │ SkillLoader    │ │ level merge  │ │                │     │
│   │ TmuxSession    │ │ validation   │ │                │     │
│   │ MCPOAuth       │ │              │ │                │     │
│   │ SkillMcpMgr    │ │              │ │                │     │
│   │ ...            │ │              │ │                │     │
│   └────────────────┘ └──────────────┘ └────────────────┘     │
│                                                                │
│   ┌────────────────────────────────────────────────────────┐  │
│   │              OpenClaw (External Integration)            │  │
│   │        Discord / Telegram / Webhook / Command           │  │
│   └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

### 2.2 Initialization Pipeline

```
pluginModule.server(input, options)
  ├─→ loadPluginConfig()         # JSONC parse → project/user merge → Zod validation → migration
  ├─→ createManagers()           # TmuxSessionManager, BackgroundManager, 
  │                              # SkillMcpManager, ConfigHandler
  ├─→ createTools()              # SkillContext + AvailableCategories + 
  │                              # ToolRegistry (26 tools)
  ├─→ createHooks()              # 3 layers: Core(43) + Continuation(7) + Skill(2) = 52
  └─→ createPluginInterface()    # 10 OpenCode Hook Handlers → PluginInterface
```

---

## 3. Agent System Details

### 3.1 Agent Directory (11)

| Agent | Model | Temperature | Mode | Fallback Chain | Positioning |
|-------|-------|-------------|------|----------------|-------------|
| **Sisyphus** | claude-opus-4-7 max | 0.1 | all | k2p5→kimi-k2.5→gpt-5.4→glm-5→big-pickle | Main orchestrator, planning + delegation |
| **Hephaestus** | gpt-5.4 medium | 0.1 | all | — | Autonomous deep worker |
| **Oracle** | gpt-5.4 high | 0.1 | subagent | gemini-3.1-pro→claude-opus-4-7 | Read-only consultant |
| **Librarian** | minimax-m2.7 | 0.1 | subagent | minimax-hs→claude-haiku-4-5→gpt-5-nano | External docs/code search |
| **Explore** | grok-code-fast-1 | 0.1 | subagent | minimax-hs→minimax-m2.7→claude-haiku→gpt-5-nano | Fast codebase Grep |
| **Multimodal-Looker** | gpt-5.3-codex | 0.1 | subagent | k2p5→gemini-3-flash→glm-4.6v→gpt-5-nano | PDF/image analysis |
| **Metis** | claude-opus-4-7 max | **0.3** | subagent | gpt-5.4 high→gemini-3.1-pro | Pre-planning consultant |
| **Momus** | gpt-5.4 xhigh | 0.1 | subagent | claude-opus-4-7→gemini-3.1-pro | Plan reviewer |
| **Atlas** | claude-sonnet-4-6 | 0.1 | primary | gpt-5.4 medium | Todo list orchestrator |
| **Prometheus** | claude-opus-4-7 max | 0.1 | — | Internal planner | Strategic planning (internal) |
| **Sisyphus-Junior** | claude-sonnet-4-6 | 0.1 | all | User configurable | Category-derived executor |

### 3.2 Agent Factory Pattern

All Agents follow a unified factory pattern:

```typescript
const createXXXAgent: AgentFactory = (model: string) => ({
  instructions: "...",
  model,
  temperature: 0.1,
  // ...config
})
createXXXAgent.mode = "subagent"  // or "primary" or "all"
```

Model resolution follows a 4-step chain: `override → category-default → provider-fallback → system-default`

### 3.3 Three Agent Modes

- **primary**: Follows UI-selected model, uses fallback chain
- **subagent**: Uses its own fallback chain, ignores UI selection
- **all**: Available in both contexts (e.g., Sisyphus-Junior)

### 3.4 Tool Restriction Matrix

| Agent | Forbidden Tools |
|-------|----------------|
| Oracle | write, edit, task, call_omo_agent |
| Librarian | write, edit, task, call_omo_agent |
| Explore | write, edit, task, call_omo_agent |
| Multimodal-Looker | All except read |
| Atlas | task, call_omo_agent |
| Momus | write, edit, task |

### 3.5 Sisyphus Dynamic Prompt System

Sisyphus's Prompt is not a static string but is **dynamically constructed** via `dynamic-agent-prompt-builder.ts`. It auto-generates based on currently available Agents, tools, skills, and categories:

- Agent identity section
- Key triggers section
- Tool selection table
- Delegation table
- Category skill guide
- Oracle/Explore/Librarian call guide
- Hard constraints and anti-patterns section

This means: **When users install new skills or configure new Agents, Sisyphus's Prompt automatically updates**.

---

## 4. Hook System Details

### 4.1 Three-Layer Hook Architecture (52)

```
createHooks()
  ├─→ createCoreHooks()           # 43
  │   ├─ createSessionHooks()     # 24
  │   │   contextWindowMonitor    # Context window monitoring
  │   │   thinkMode               # Think mode
  │   │   ralphLoop               # Ralph loop
  │   │   modelFallback           # Model fallback
  │   │   runtimeFallback         # Runtime fallback
  │   │   anthropicEffort         # Anthropic effort level
  │   │   intentGate              # Intent classification gate
  │   │   legacyPluginToast       # Legacy plugin toast
  │   │   ...
  │   ├─ createToolGuardHooks()   # 14
  │   │   commentChecker          # AI nonsense comment detection
  │   │   rulesInjector           # Rules injection
  │   │   writeExistingFileGuard  # Write protection
  │   │   hashlineReadEnhancer    # Hashline read enhancement
  │   │   bashFileReadGuard       # Bash file read guard
  │   │   readImageResizer        # Image resize on read
  │   │   ...
  │   └─ createTransformHooks()   # 5
  │       claudeCodeHooks         # Claude Code compatibility
  │       keywordDetector         # Keyword detection
  │       contextInjector         # Context injection
  │       thinkingBlockValidator  # Thinking block validation
  │       toolPairValidator       # Tool pair validation
  ├─→ createContinuationHooks()   # 7
  │   todoContinuationEnforcer    # Todo continuation enforcer
  │   atlas                       # Atlas orchestration
  │   stopContinuationGuard       # Stop guard
  │   compactionContextInjector   # Compaction context injection
  │   ...
  └─→ createSkillHooks()          # 2
      categorySkillReminder       # Category skill reminder
      autoSlashCommand            # Auto slash command
```

### 4.2 Notable Hook Mechanisms

**IntentGate**: Classifies user intent (research/implementation/investigation/evaluation/fix) before the Agent responds, selecting different processing paths based on intent type.

**Ralph Loop**: Self-referential loop mechanism where the Agent continuously works until the task is fully completed without manual intervention.

**Todo Continuation Enforcer**: When the Agent is idle, the system automatically pulls it back to pending tasks — ensuring tasks will definitely be completed.

**Hashline Read Enhancer**: Appends content hash tags to each line read (`LINE#ID`), verifying hash match during edits — preventing stale line edits.

**Comment Checker**: Detects and blocks AI-generated nonsense comment patterns (e.g., "This function does X" type of valueless comments).

---

## 5. Tool System Details

### 5.1 Tool Inventory (26)

| Directory | Tool | Purpose |
|-----------|------|---------|
| `ast-grep/` | AST-Grep Search/Replace | AST-aware code search and rewrite (25 languages) |
| `background-task/` | Background Task | Async background task management |
| `call-omo-agent/` | Call OMO Agent | Call internal Agents |
| `delegate-task/` | Delegate Task | Category-based task delegation (8 built-in categories) |
| `glob/` | Glob | File pattern matching search |
| `grep/` | Grep | Regex content search |
| `hashline-edit/` | Hashline Edit | **LINE#ID hash-anchored editing** |
| `interactive-bash/` | Interactive Bash (Tmux) | Full interactive terminal (REPL/debuggers/TUI) |
| `look-at/` | Look At | Multimodal file analysis (PDF/images/charts) |
| `lsp/` | LSP Rename/Goto/References/Diagnostics/Symbols | IDE-precision code navigation and diagnostics |
| `session-manager/` | Session List/Read/Search/Info | Session history management |
| `shared/` | Shared tool infrastructure | — |
| `skill-mcp/` | Skill MCP | Skill-embedded MCP invocation |
| `skill/` | Skill Loader | Skill loader |
| `slashcommand/` | Slash Commands | `/init-deep`, `/start-work` etc. |
| `task/` | Task (Subagent) | Sub-Agent dispatch and management |
| `index.ts` | Tool registry entry | — |

### 5.2 Hashline Edit Tool — Solving the Harness Problem

This is one of OmO's most innovative tools, inspired by [oh-my-pi](https://github.com/can1357/oh-my-pi):

**Problem**: Traditional editing tools require the AI to reproduce file content to locate edit positions, but AI often cannot reproduce precisely, causing "stale line" errors.

**Solution**: Each time a file is read, append a content hash to each line:

```
11#VK| function hello() {
22#XJ|   return "world";
33#MB| }
```

Agents reference these tags to edit. If the file was modified after reading, hashes won't match and the edit is rejected. **Grok Code Fast 1's success rate increased from 6.7% to 68.3%**, solely by changing the editing tool.

### 5.3 Delegate Task System

8 built-in categories, each auto-routed to the most suitable model:

| Category | Purpose |
|----------|---------|
| `visual-engineering` | Frontend, UI/UX, design |
| `deep` | Autonomous research + execution |
| `quick` | Single file changes, small modifications |
| `ultrabrain` | Hard logic, architecture decisions |
| `artistry` | Unconventional creative solutions |
| `unspecified-low` | Low complexity misc |
| `unspecified-high` | High complexity misc |
| `writing` | Documentation, technical writing |

---

## 6. MCP Three-Tier System

| Tier | Source | Mechanism |
|------|--------|-----------|
| **Tier 1: Built-in** | `src/mcp/` | 3 remote HTTP MCPs: websearch (Exa/Tavily), context7, grep_app |
| **Tier 2: Claude Code** | `.mcp.json` | Environment variable expansion via claude-code-mcp-loader |
| **Tier 3: Skill-embedded** | SKILL.md YAML | Managed by SkillMcpManager (stdio + HTTP) |

Skill-embedded MCP is a unique innovation: skills come with their own MCP servers, started on demand and destroyed after task completion, not wasting context window space.

---

## 7. Configuration System

### 7.1 Multi-Level Config Merge

```
Project (.opencode/oh-my-opencode.jsonc)
    ↓ merge
User (~/.config/opencode/oh-my-opencode.jsonc)
    ↓ merge
Defaults (Zod schema)
```

Merge strategy:
- `agents`, `categories`, `claude_code`: **Deep recursive merge** (prototype pollution safe)
- `disabled_*` arrays: **Set union** (deduplicated concatenation)
- Other fields: **Override replacement**

### 7.2 Config Validation

Uses **Zod v4** for runtime validation, `safeParse()` fills defaults for omitted fields. Config migration is idempotent via `_migrations` tracking.

---

## 8. OpenClaw — External Integration System

`src/openclaw/` implements bidirectional external integration:

- **Discord**: Receive/send messages via Bot
- **Telegram**: Receive/send messages via Bot
- **Webhook**: HTTP callback notifications
- **Command**: Command line interface

This allows OmO to function as a continuously running AI development assistant, interacting with users through multiple channels.

---

## 9. Build and Distribution

### 9.1 Build Pipeline

```bash
bun build src/index.ts --outdir dist --target bun --format esm   # ESM build
tsc --emitDeclarationOnly                                         # Type declarations
bun build src/cli/index.ts --outdir dist/cli --target bun        # CLI build
bun run build:schema                                              # JSON Schema generation
```

### 9.2 Platform Binaries

11 platform-specific pre-compiled binaries (via `bun compile`):

| Platform | Architecture | Variant |
|----------|-------------|---------|
| darwin | arm64, x64 | AVX2 + baseline |
| linux | arm64, x64 | AVX2 + baseline + musl |
| windows | x64 | AVX2 + baseline |

Runtime auto-detects AVX2 and libc type, falling back to baseline version.

### 9.3 Dual Package Publishing

Published under both `oh-my-opencode` and `oh-my-openagent` names on npm (transition period).

---

## 10. Feature Deep Dive

### 10.1 ultrawork Mode

Users simply type `ultrawork` (or `ulw`), and the system automatically:
1. Activates all Agents
2. Starts Ralph Loop
3. Dispatches background tasks in parallel
4. Continuously works until completion

### 10.2 Prometheus Planning System

The `/start-work` command invokes the Prometheus Agent in interviewer mode:
1. Asks questions to identify scope
2. Discovers ambiguities
3. Builds a validated plan
4. Only then starts coding

### 10.3 /init-deep Command

Automatically generates hierarchical `AGENTS.md` files throughout the project:

```
project/
├── AGENTS.md              ← Project-level context
├── src/
│   ├── AGENTS.md          ← src-specific context
│   └── components/
│       └── AGENTS.md      ← Component-specific context
```

Agents automatically read relevant context layers — zero manual management.

### 10.4 Doctor Diagnostics

`bunx oh-my-opencode doctor` performs health checks:
- Plugin registration verification
- Config validation
- Model availability check
- Environment check

---

## 11. Engineering Standards

### 11.1 Code Standards

- **Runtime**: Bun only (1.3.11), npm/yarn disabled
- **TypeScript**: strict mode, ESNext, bundler moduleResolution
- **Types**: `bun-types`, `@types/node` disabled
- **File naming**: kebab-case
- **Module structure**: index.ts barrel exports, no catch-all files
- **200 LOC soft limit**: Each file should not exceed 200 lines
- **Relative imports**: `@/` path aliases forbidden
- **Factory pattern**: All tools, Hooks, Agents use `createXXX()` factories

### 11.2 Anti-Patterns (Enforced)

- No `as any`, `@ts-ignore`, `@ts-expect-error`
- No empty catch blocks
- No AI-generated comments (enforced by comment-checker Hook)
- No emojis (unless explicitly requested by user)
- No business logic in index.ts

### 11.3 Testing Standards

- **Framework**: Bun test (`bun:test`)
- **Style**: given/when/then pattern
- **Location**: Same directory `*.test.ts`
- **CI splitting**: Auto-isolates tests using `mock.module()`

---

## 12. Strengths and Limitations

### Strengths

1. **Full-stack Agent orchestration**: 11 specialized Agents covering planning to execution
2. **Multi-model support**: Not locked to a single model, each Agent matched to optimal model
3. **Rich toolset**: LSP, AST-Grep, Hashline editing, Tmux and other IDE-level tools
4. **Hook depth**: 52 lifecycle Hooks covering almost every execution stage
5. **Claude Code compatible**: Complete Hook, Command, Skill, MCP, Plugin compatibility layer
6. **Skill-embedded MCP**: Innovatively solves MCP context bloat problem
7. **Hashline editing**: Fundamentally solves Agent editing tool reliability issues
8. **Production-grade quality**: 377k LOC, 104 barrel exports, strict type system

### Limitations

1. **High complexity**: 1,766 source files create a barrier to understanding and contribution
2. **OpenCode dependency**: Core functionality bound to @opencode-ai/plugin API
3. **Multi-model cost**: Using multiple commercial APIs simultaneously may incur significant costs
4. **Platform-specific**: Some features (e.g., Tmux) depend on specific platform support
5. **SUL-1.0 license**: Non-standard open source license, commercial use requires attention
