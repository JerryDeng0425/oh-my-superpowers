# oh-my-superpowers Skills Native Installation Feasibility Analysis

> **Analysis Date**: 2026-04-19  
> **Question**: Can oh-my-superpowers skills be natively discovered and used by OpenCode?

---

## 1. Conclusion

**Yes.** oh-my-superpowers already has the complete infrastructure for native OpenCode loading without any code modifications. However, the reference paths in the plugin JS need to be updated to match the `omo-` prefix renaming.

---

## 2. How It Works: OpenCode Skill Discovery Mechanism

### 2.1 Six-Layer Skill Discovery Sources (Priority High to Low)

OpenCode discovers `SKILL.md` files from 6 locations via `discoverSkills()`:

```
Priority 1: opencode-project    ← Project directory/.opencode/skills/
Priority 2: opencode            ← ~/.config/opencode/skills/
Priority 3: project             ← Project directory/.claude/skills/ + .agents/skills/
Priority 4: agents-project      ← Project directory/.agents/skills/  
Priority 5: user                ← ~/.claude/skills/ (Claude Code config directory)
Priority 6: agents-global       ← ~/.agents/skills/
```

**Same-name skills**: Higher priority overrides lower priority (`deduplicateSkillsByName`).

### 2.2 Skill File Format Requirements

OpenCode's `loadSkillsFromDir()` looks for two file patterns:

```typescript
// Primary lookup
skills/omo-brainstorming/SKILL.md          ← directory-name/SKILL.md

// Fallback lookup
skills/omo-brainstorming/brainstorming.md   ← directory-name.md
```

Skill file format (`SKILL.md`):

```markdown
---
name: omo-brainstorming           # Skill name (required)
description: Purpose description   # Skill description (required)
model: claude-opus-4-7            # Optional: specify model
agent: sisyphus                   # Optional: specify Agent
subtask: true                     # Optional: mark as subtask
allowed-tools: [Bash, Read]       # Optional: allowed tools
mcp:                              # Optional: embedded MCP config
  - name: my-mcp
    type: stdio
    command: npx
    args: [-y, my-mcp-server]
---

Skill content (instructions for the Agent)...
```

### 2.3 Superpowers Plugin Registration Mechanism

`oh-my-superpowers/.opencode/plugins/superpowers.js` does two things:

**1. Config Hook — Inject Skill Path**

```javascript
config: async (config) => {
  config.skills = config.skills || {};
  config.skills.paths = config.skills.paths || [];
  if (!config.skills.paths.includes(superpowersSkillsDir)) {
    config.skills.paths.push(superpowersSkillsDir);
  }
}
```

This injects the `skills/` directory into OpenCode's config, making it one of the skill discovery paths.

**2. Messages Transform Hook — Inject Bootstrap Context**

```javascript
'experimental.chat.messages.transform': async (_input, output) => {
  // Reads omo-using-superpowers/SKILL.md (but currently has hardcoded old path)
  // Injects before first user message
}
```

---

## 3. Current Issue Analysis

### 3.1 Hardcoded Path Issue

`superpowers.js` line 58:

```javascript
const skillPath = path.join(superpowersSkillsDir, 'using-superpowers', 'SKILL.md');
```

**Issue**: Skill folder has been renamed to `omo-using-superpowers`, but `superpowers.js` still references `using-superpowers`.

**Impact**: Bootstrap context injection fails, `getBootstrapContent()` returns `null`, skill discovery mechanism won't be auto-injected at session start.

### 3.2 Old References in Bootstrap Text

`superpowers.js` lines 64-71 tool mapping text still references old skill name formats.

### 3.3 `package.json` `main` Field

```json
{ "main": ".opencode/plugins/superpowers.js" }
```

This is correct and doesn't need modification.

---

## 4. Required Changes

### Change 1: Path in `superpowers.js` (Required)

```diff
- const skillPath = path.join(superpowersSkillsDir, 'using-superpowers', 'SKILL.md');
+ const skillPath = path.join(superpowersSkillsDir, 'omo-using-superpowers', 'SKILL.md');
```

### Change 2: Bootstrap Text in `superpowers.js` (Required)

```diff
- You have superpowers.
+ You have superpowers (oh-my-opencode integrated).

- **IMPORTANT: The using-superpowers skill content is included below.**
+ **IMPORTANT: The omo-using-superpowers skill content is included below.**

- Do NOT use the skill tool to load "using-superpowers" again
+ Do NOT use the skill tool to load "omo-using-superpowers" again
```

### Change 3: `package.json` Name (Optional but Recommended)

```diff
- "name": "superpowers",
+ "name": "oh-my-superpowers",
```

---

## 5. Installation Methods

### Method A: Via `opencode.json` Plugin Declaration (Recommended)

Add to project's `opencode.json` or global `~/.config/opencode/opencode.json`:

```json
{
  "plugin": ["oh-my-superpowers@git+https://github.com/<your-username>/oh-my-superpowers.git"]
}
```

**How it works**:
1. OpenCode reads `plugin` config at startup
2. Clones/pulls Git repository
3. Executes `package.json`'s `main` field pointing to `.opencode/plugins/superpowers.js`
4. Plugin's `config` hook injects `skills/` path
5. OpenCode's `discoverSkills()` scans all `SKILL.md` under that path
6. Plugin's `messages.transform` hook injects bootstrap context

### Method B: Manual Copy to OpenCode Skill Directory

```bash
# Global install (all projects)
cp -r oh-my-superpowers/skills/* ~/.config/opencode/skills/

# Or project-level install (current project only)
mkdir -p .opencode/skills/
cp -r oh-my-superpowers/skills/* .opencode/skills/
```

**Pros**: No plugin mechanism needed, pure file copy  
**Cons**: Loses `superpowers.js` bootstrap injection, needs manual skill discovery trigger

### Method C: Via oh-my-opencode Config's skills.paths

Configure in `oh-my-opencode.jsonc`:

```jsonc
{
  "skills": {
    "paths": ["/absolute/path/to/oh-my-superpowers/skills"]
  }
}
```

---

## 6. Complete Installation Flow (Method A Recommended)

### Step 1: Fix `superpowers.js`

Modify path references in `oh-my-superpowers/.opencode/plugins/superpowers.js` (see Changes 1 and 2 above).

### Step 2: Push to Git Repository

```bash
cd oh-my-superpowers
git init && git add . && git commit -m "oh-my-superpowers: omo- prefixed skills with OmO integration"
git remote add origin https://github.com/<your-username>/oh-my-superpowers.git
git push -u origin main
```

### Step 3: Configure OpenCode

```json
// ~/.config/opencode/opencode.json
{
  "plugin": ["oh-my-superpowers@git+https://github.com/<your-username>/oh-my-superpowers.git"]
}
```

### Step 4: Verify

```
# In OpenCode
use skill tool to list skills
# Should see 14 omo- prefixed skills
```

---

## 7. Compatibility Matrix

| Skill Format Feature | oh-my-superpowers | OpenCode Expects | Compatible? |
|---------------------|-------------------|-----------------|-------------|
| Directory structure `skill-name/SKILL.md` | `omo-xxx/SKILL.md` ✅ | Same | ✅ |
| YAML frontmatter `name` field | `name: omo-xxx` ✅ | Same | ✅ |
| YAML frontmatter `description` field | ✅ Present | Same | ✅ |
| Markdown body as Prompt | ✅ Correct | Same | ✅ |
| Nested directory recursive scanning | `references/` subdirectory | Supports `maxDepth=2` | ✅ |
| MCP config (frontmatter) | Not used | Optional | ✅ No impact |
| `allowed-tools` field | Not used | Optional | ✅ No impact |
| Plugin JS (config hook) | ✅ Present | OpenCode plugin API | ⚠️ Needs path fix |
| Plugin JS (messages transform) | ✅ Present | OpenCode plugin API | ⚠️ Needs path fix |

---

## 8. Post-Installation Workflow

```
User starts OpenCode session
  │
  ├─→ superpowers.js config hook executes
  │     └─→ Injects skills/ path into config.skills.paths
  │
  ├─→ OpenCode skill discovery engine executes
  │     └─→ Scans skills/ directory → discovers 14 omo-xxx/SKILL.md
  │     └─→ Parses YAML frontmatter → registers 14 skills
  │
  ├─→ superpowers.js messages.transform hook executes
  │     └─→ Reads omo-using-superpowers/SKILL.md
  │     └─→ Injects before first user message
  │
  ├─→ oh-my-opencode's categorySkillReminder hook executes
  │     └─→ Checks Sisyphus's available skill list
  │     └─→ Injects 14 omo- skills into Agent Prompt
  │
  └─→ Sisyphus sees skill list
        └─→ Auto-selects appropriate skills based on task type
        └─→ task() calls in skills route to OmO Agents
```

---

## 9. Risks and Considerations

1. **oh-my-opencode already has same-name built-in skills**: OmO's built-in skills (e.g., `git-master`, `playwright`, `frontend-ui-ux`) don't overlap with oh-my-superpowers skills (`omo-` prefix), so there's no conflict.

2. **`task()` call format in skill content**: When OmO skills are loaded via Skill Tool, `task()` calls need to match OmO's `delegate-task` tool interface. Our Path 2 integration already handles this.

3. **Skill size**: 14 skill files total approximately 100KB, won't significantly impact context window (skills are loaded on-demand, not all injected at once).

4. **Version management**: Using Git URL installation allows pinning to specific tag/branch:
   ```json
   "plugin": ["oh-my-superpowers@git+https://github.com/xxx/oh-my-superpowers.git#v5.0.7-omo.1"]
   ```
