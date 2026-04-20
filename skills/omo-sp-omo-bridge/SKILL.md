---
name: omo-sp-omo-bridge
description: Bridge between Superpowers workflow roles and oh-my-opencode agents. Automatically maps SP roles to OmO agents. Loads before any skill dispatches sub-agents.
---

# SP → OMO Agent Bridge

When Superpowers workflow dispatches a sub-agent role, use this mapping to route to the correct oh-my-opencode agent.

## Role → Agent Mapping

| SP Role | OMO Tool | Parameters |
|---------|----------|------------|
| Implementer (simple) | `task()` | `category="quick", load_skills=["omo-test-driven-development"]` |
| Implementer (complex) | `task()` | `category="deep", load_skills=["omo-test-driven-development"]` |
| Implementer (hard logic) | `task()` | `category="ultrabrain", load_skills=["omo-test-driven-development"]` |
| Implementer (frontend) | `task()` | `category="visual-engineering", load_skills=["omo-test-driven-development", "frontend-ui-ux"]` |
| Spec Reviewer | `task()` | `subagent_type="momus"` |
| Code Quality Reviewer | `task()` | `subagent_type="oracle"` |
| Codebase Search | `task()` | `subagent_type="explore", run_in_background=true` |
| External Research | `task()` | `subagent_type="librarian", run_in_background=true` |
| Pre-planning Analysis | `task()` | `subagent_type="metis"` |
| Plan Creation | `task()` | `subagent_type="prometheus"` or `/start-work` command |
| Plan Review | `task()` | `subagent_type="momus"` |

## Automatic Skill Injection

When dispatching implementation tasks, ALWAYS inject relevant skills:

| Task Type | load_skills |
|-----------|-------------|
| Any implementation | `["omo-test-driven-development"]` |
| Bug fix | `["omo-test-driven-development", "omo-systematic-debugging"]` |
| Feature work | `["omo-test-driven-development", "omo-verification-before-completion"]` |
| Frontend | `["omo-test-driven-development", "frontend-ui-ux"]` |
| Git operations | `["git-master"]` |

## Two-Stage Review (Superpowers Discipline)

When executing subagent-driven-development workflow:

1. After implementer DONE → dispatch `subagent_type="momus"` with spec compliance prompt
2. After spec OKAY → dispatch `subagent_type="oracle"` with code quality prompt
3. If either review finds issues → continue implementer session with `session_id`
4. Never skip either review stage

## Three-Stage Planning Pipeline

For complex features, use the full planning pipeline:

1. **Metis** (pre-analysis) → Detects ambiguities, classifies intent, identifies AI failure points
2. **Prometheus** (planning) → Interviews user, explores codebase, builds detailed implementation plan
3. **Momus** (plan review) → Verifies plan executability, checks references, validates completeness

## Session Continuation

Always use `session_id` to continue a sub-agent's work:

```
// First dispatch → get session_id in result
task(category="quick", load_skills=["omo-test-driven-development"], description="Task 1", prompt="...", run_in_background=false)
// Result includes session_id: "ses_abc123"

// Continue same agent with full context preserved
task(session_id="ses_abc123", load_skills=["omo-test-driven-development"], description="Fix issues in Task 1", prompt="Fix: ...", run_in_background=false)
```

## Background Dispatch for Parallel Work

Independent exploration/research tasks should run in background:

```
task(subagent_type="explore", description="Find auth patterns", prompt="...", run_in_background=true)
task(subagent_type="librarian", description="Research JWT best practices", prompt="...", run_in_background=true)
// Continue working → collect results later with background_output(task_id="...")
```
