# Browser Automation Skill

A Claude Code skill that delegates browser automation tasks to subagents, keeping the main agent's context window clean.

## Installation

```bash
# Using skills CLI (recommended)
npx skills add hienlh/claude-skill-browser-automation

# Global install, no prompts
npx skills add hienlh/claude-skill-browser-automation -y -g

# Manual install
git clone https://github.com/hienlh/claude-skill-browser-automation.git ~/.claude/skills/browser-automation
```

## Features

- **Context-efficient**: ~100 tokens vs ~3000 tokens per browser task
- **Interactive flow**: Supports login/auth with user input (NEEDS_INPUT protocol)
- **Auto-reload**: Handles stuck pages, frozen UI, unresponsive actions
- **Optimized patterns**: Uses `browser_run_code` for batching actions

## Usage

```javascript
Task({
  subagent_type: "general-purpose",
  description: "Browser: [description]",
  prompt: `BROWSER TASK: [what to do]
    Use Playwright MCP. Prefer browser_run_code.
    Return: SUCCESS/FAILED/NEEDS_INPUT + AgentId if pausing.`
})
```

### Interactive Flow (Login/Auth)

```
1. Subagent returns:    Status: NEEDS_INPUT | Fields: email, password | AgentId: abc123
2. Main agent:          AskUserQuestion for credentials
3. Resume subagent:     Task({ resume: "abc123", prompt: "email: x, password: y" })
```

## Files

```
browser-automation/
├── SKILL.md                      # Quick reference
└── references/
    ├── interactive-flow.md       # Login/auth handling
    ├── examples.md               # Code examples
    └── troubleshooting.md        # Error handling & stuck detection
```

## Requirements

- Claude Code with Playwright MCP configured
- Task tool access for subagent delegation

## Version

1.3.1

## License

MIT
