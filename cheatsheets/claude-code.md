# Claude Code Cheat Sheet

_Last updated: 2026-04-07_

**Purpose:** AI-powered coding assistant in the terminal and IDE.  
**Assumptions:** Installed via `curl -fsSL https://claude.ai/install.sh | bash` or `brew install --cask claude-code`. Git ≥ 2.23.

---

## Quick start

```sh
claude                          # start interactive session
claude "query"                  # start with a prompt
claude -p "query"               # non-interactive (print mode), then exit
claude -c                       # continue most recent conversation
claude -r "SESSION"             # resume a specific session
claude update                   # update to latest version
claude doctor                   # verify installation & config
claude auth login               # sign in
```

---

## Keyboard shortcuts (interactive)

| Key | Action |
|-----|--------|
| `Ctrl+C` | Cancel / interrupt |
| `Ctrl+D` | Exit |
| `Ctrl+L` | Redraw screen |
| `Ctrl+R` | Reverse history search |
| `Ctrl+T` | Toggle task list |
| `Ctrl+B` | Background running task |
| `Ctrl+G` | Open input in external editor |
| `Shift+Tab` / `Alt+M` | Cycle permission modes |
| `Alt+P` | Switch model |
| `Alt+T` | Toggle extended thinking |
| `Alt+O` | Toggle fast mode |
| `Esc Esc` | Rewind / summarize |

Multiline input: `Option+Enter` (macOS), `Shift+Enter` (iTerm2/Ghostty/Kitty/WezTerm), `\ + Enter` (any terminal).

Special prefixes at the start of input:

- `/` — slash commands and skills menu
- `!` — run a shell command directly
- `@` — file path autocomplete

---

## Slash commands

### Session

| Command | Purpose |
|---------|---------|
| `/help` | Show help |
| `/clear` | Start fresh session |
| `/compact` | Compress context window (use when conversation gets long) |
| `/resume` | Resume a previous session |
| `/export` | Save conversation to text file |
| `/cost` | View token usage / cost |
| `/stats` | Visualize daily usage and session history |
| `/status` | Show version, model, account, connectivity |

### Configuration

| Command | Purpose |
|---------|---------|
| `/config` | Open settings (model, theme, permissions) |
| `/init` | Generate or update CLAUDE.md for a project |
| `/memory` | View / edit CLAUDE.md and auto-memory |
| `/permissions` | Manage tool allow / deny rules |
| `/terminal-setup` | Configure terminal keybindings (Shift+Enter, etc.) |
| `/vim` | Toggle vim mode |
| `/mcp` | View / manage MCP servers and OAuth |
| `/hooks` | View configured hooks |

### Workflows (built-in skills)

| Command | Purpose |
|---------|---------|
| `/review` | Review a pull request |
| `/simplify` | Review changed code for quality, then fix |
| `/simplify <focus>` | Focus review (e.g., `/simplify error handling`) |
| `/batch <instruction>` | Parallel changes across 5-30 worktrees (auto-PRs) |
| `/loop [interval] <prompt>` | Repeat a prompt every N minutes (default 10m) |
| `/schedule` | Create / manage cron-style remote agents |
| `/claude-api` | Load Claude API + SDK reference for your language |
| `/install-github-app` | Set up Claude GitHub Actions integration |

---

## CLI flags

```sh
# Prompt & session
-p, --print                     # non-interactive mode
-c, --continue                  # continue last conversation
-r, --resume "SESSION"          # resume by name/ID
-n, --name "NAME"               # name the session

# Model & effort
--model MODEL                   # use specific model (opus, sonnet, haiku)
--effort low|medium|high|max    # reasoning effort level

# Permissions
--permission-mode MODE          # default|plan|acceptEdits|auto|dontAsk
--allowedTools "TOOL_PATTERN"   # auto-approve specific tools
--disallowedTools "TOOL_PATTERN"

# Output
--output-format text|json|stream-json
--json-schema 'SCHEMA'          # structured output
--verbose                       # full turn-by-turn output

# Limits
--max-turns N                   # max agentic turns
--max-budget-usd AMOUNT         # spending cap

# Isolation & browser
-w, --worktree "NAME"           # run in isolated git worktree (clean branch)
--chrome                        # enable browser integration (screenshots, testing)

# Startup
--bare                          # skip auto-discovery (fast startup)
--add-dir PATH                  # add extra working directories
--system-prompt "TEXT"          # replace default system prompt
--append-system-prompt "TEXT"   # add to default system prompt
```

---

## Configuration files

```
~/.claude/
  settings.json                 # user-wide settings
  CLAUDE.md                     # user-wide instructions
  keybindings.json              # keyboard shortcuts
  .mcp.json                     # user-wide MCP servers
  skills/                       # personal custom skills

<project>/
  CLAUDE.md                     # project instructions (committed)
  CLAUDE.local.md               # personal overrides (gitignored)
  .claudeignore                 # ignore patterns (like .gitignore)
  .claude/
    settings.json               # project settings (committed)
    settings.local.json         # local project settings (gitignored)
    .mcp.json                   # project MCP servers
    rules/*.md                  # path-specific rules
    agents/*.md                 # subagent definitions
    skills/                     # project custom skills
    hooks/                      # hook scripts
```

---

## Permission modes

| Mode | Behavior |
|------|----------|
| `default` | Prompts for each action |
| `plan` | Describe changes first, then ask |
| `acceptEdits` | Auto-approve file edits, prompt for commands |
| `auto` | Smart classifier (Team/Enterprise) |
| `dontAsk` | Deny anything not in allow rules |

Switch modes: `Shift+Tab` in session, or `--permission-mode` flag, or `/permissions`.

Rule patterns: `Bash(git *)`, `Edit(*.js)`, `Edit(src/**)`, `Read`.

---

## MCP servers

```sh
claude mcp add --transport stdio NAME COMMAND
claude mcp add --transport http NAME URL
claude mcp list
claude mcp remove NAME
```

Or configure in `.claude/.mcp.json`:

```json
{
  "mcpServers": {
    "name": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-example"],
      "env": { "TOKEN": "..." }
    }
  }
}
```

### Connectors (pre-built via claude.ai)

Configured at [claude.ai/settings/connectors](https://claude.ai/settings/connectors), automatically available in Claude Code. Common connectors:

- **Gamma** — presentations, documents, webpages, social posts
- **Atlassian** — Jira issues, Confluence pages, search, comments
- **Microsoft 365** — Outlook email/calendar, SharePoint, meeting availability

Invoked naturally in conversation ("create a presentation about X"). Manage with `/mcp`.

---

## Hooks

Hooks run shell commands or prompts in response to events. Configure in `settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "npx prettier --write $CLAUDE_FILE_PATH"
      }]
    }]
  }
}
```

Key events: `PreToolUse` (can block), `PostToolUse`, `SessionStart`, `UserPromptSubmit`, `Stop`.

Create hooks from natural language with the hookify plugin: `/hookify never edit files in vendor/`.

---

## Official plugins

Install with `/plugin`. Source: [github.com/anthropics/claude-code/tree/main/plugins](https://github.com/anthropics/claude-code/tree/main/plugins).

| Plugin | Commands | How it works |
|--------|----------|-------------|
| **frontend-design** | (auto-triggers) | Ask to build web UI; generates production-grade, distinctive designs |
| **feature-dev** | `/feature-dev <desc>` | Structured 7-phase feature development with agents |
| **code-review** | `/code-review` | 5 parallel agents review current PR |
| **pr-review-toolkit** | `/pr-review-toolkit:review-pr [areas]` | Deep review: tests, errors, types, simplify, all |
| **security-guidance** | (auto-hook) | Warns on security anti-patterns (XSS, injection, eval) |
| **commit-commands** | `/commit`, `/commit-push-pr`, `/clean_gone` | Streamlined git workflow |
| **hookify** | `/hookify <rule>`, `/hookify:list` | Create hooks from natural language |
| **ralph-wiggum** | `/ralph-loop`, `/cancel-ralph` | Autonomous loop until task completion |
| **plugin-dev** | `/plugin-dev:create-plugin` | Guided workflow for building plugins |
| **agent-sdk-dev** | `/new-sdk-app` | Scaffold new Claude Agent SDK project |

---

## Custom skills

Define your own in `~/.claude/skills/` (all projects) or `.claude/skills/` (project-only).

Each skill is a directory with a `SKILL.md` file:

```yaml
---
name: my-skill
description: When and why to use this skill
allowed-tools: Read Grep Bash   # auto-approved tools (optional)
model: sonnet                   # override model (optional)
context: fork                   # isolated subagent context (optional)
---

Your instructions here. Use $ARGUMENTS for user input.
```

Invoke: `/my-skill <args>`. Claude can also auto-trigger based on context.

**Tips for effective skills:**

- Be specific about role: "Review as a security engineer focused on OWASP Top 10"
- Use tight guardrails for critical tasks (migrations, deployments, schema changes)
- Use `context: fork` for heavy exploration (keeps main context clean)
- Test across scenarios before relying on a skill

Community plugins and marketplace: `/plugin` to browse.

---

## Environment variables

```sh
ANTHROPIC_API_KEY               # API key
ANTHROPIC_BASE_URL              # proxy endpoint
ANTHROPIC_MODEL                 # default model
CLAUDE_CODE_USE_BEDROCK         # use Amazon Bedrock
CLAUDE_CODE_EFFORT_LEVEL        # low/medium/high/max
DISABLE_AUTOUPDATER             # disable auto-updates
```

Set via `settings.json` `"env"` block or export in shell.

---

## References

- Docs: [docs.anthropic.com/en/docs/claude-code](https://docs.anthropic.com/en/docs/claude-code)
- GitHub: [github.com/anthropics/claude-code](https://github.com/anthropics/claude-code)
- Plugins: [github.com/anthropics/claude-code/tree/main/plugins](https://github.com/anthropics/claude-code/tree/main/plugins)
