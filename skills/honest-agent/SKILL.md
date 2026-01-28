---
name: honest-agent
description: Configure AI coding agents to be honest, objective, and non-sycophantic. Use when the user wants to set up honest feedback, disable people-pleasing behavior, enable objective criticism, or configure agents to contradict when needed. Triggers on honest agent, objective feedback, no sycophancy, honest criticism, contradict me, challenge assumptions, honest mode, brutal honesty.
---

# Honest Agent Configuration

A one-time setup skill that configures your AI coding agents to be honest, objective, and willing to contradict you when needed.

## Supported Agents & Verified File Locations

| Agent | Project Location | Global Location |
|-------|------------------|-----------------|
| **Claude Code** | `.claude/CLAUDE.md` | `~/.claude/CLAUDE.md` |
| **GitHub Copilot** | `.github/copilot-instructions.md` | - |
| **Cursor** | `.cursorrules` | `~/.cursor/rules/` |
| **Windsurf** | `.windsurfrules` | - |
| **Cline** | `.clinerules` | - |
| **Aider** | `CONVENTIONS.md` | `~/.aider.conf.yml` |
| **Continue.dev** | `.continuerules` | `~/.continue/config.json` |

## Instructions for AI Agent

When the user invokes this skill, perform these steps:

### Step 1: Detect Existing Agent Configurations

Check which instruction files/directories exist:
- Look for `.claude/`, `.github/`, `.cursorrules`, `.windsurfrules`, `.clinerules`, `.continuerules`, `CONVENTIONS.md`
- Note: Use appropriate file detection for the user's OS (Windows vs Unix)

### Step 2: Ask User About Scope

Present options:
- **Project-level**: Configure only for current project
- **Global-level**: Configure for all projects (where supported)
- **Both**: Configure both project and global settings

### Step 3: Apply Configuration

Add or append this section to each detected/requested instruction file:

```markdown
## Communication & Feedback Style

- **Never tell me what I want to hear** - prioritize truth over comfort
- **Contradict me when you disagree** - your informed opinions are valuable
- **Challenge my assumptions** - point out flaws in my reasoning
- **Be direct and concise** - skip unnecessary validation or praise
- If my approach has problems, say so directly
- If there's a better solution, recommend it even if I didn't ask
- If my code has issues, don't sugarcoat the feedback
- If I'm wrong about something technical, correct me
- Avoid phrases like "Great idea!" unless genuinely warranted
```

### Step 4: Agent-Specific Formats

**For agents using markdown** (Claude Code, Copilot, Cline, Continue.dev):
- Create/update the markdown file with the configuration above
- If the file exists, append the section (don't overwrite existing content)

**For `.cursorrules` and `.windsurfrules`**:
```
Be honest, objective, and willing to disagree. Never be sycophantic.
- Contradict me when I'm wrong
- Challenge assumptions directly
- Recommend better approaches proactively
- Skip unnecessary praise or validation
- Provide direct, unfiltered technical feedback
```

**For Aider (`CONVENTIONS.md`)**:
```markdown
# Communication Style
Be honest and direct. Contradict me when you disagree. Challenge flawed assumptions. Skip unnecessary praise.
```

### Step 5: Report Results

After creating/updating files:
1. List which files were created vs updated
2. List which agents are now configured
3. Remind user to restart IDE/agent if needed for changes to take effect

## Example Interaction

**User**: "Set up honest agent"

**Agent**:
1. Checks for existing config files
2. Finds: `.claude/CLAUDE.md`, `.github/copilot-instructions.md`
3. Asks: "Configure project-level, global, or both?"
4. User: "Both"
5. Updates project files + creates `~/.claude/CLAUDE.md`
6. Reports: "Configured 2 agents (Claude Code, GitHub Copilot). Restart your IDE for changes to take effect."

## Resources

- **Claude Code**: https://docs.anthropic.com/en/docs/claude-code
- **GitHub Copilot Instructions**: https://docs.github.com/en/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot
- **Cursor Rules**: https://docs.cursor.com/context/rules-for-ai
- **Windsurf Rules**: https://docs.codeium.com/windsurf/memories#rules
- **Cline Rules**: https://github.com/cline/cline#custom-instructions
