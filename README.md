# Browser Automation Skill

A Claude Code skill that delegates browser automation tasks to subagents, keeping the main agent's context window clean.

## Features

- **Context-efficient**: ~100 tokens vs ~3000 tokens per browser task
- **Interactive flow**: Supports login/auth with user input (NEEDS_INPUT protocol)
- **Auto-reload**: Handles stuck pages, frozen UI, unresponsive actions
- **Optimized patterns**: Uses `browser_run_code` for batching actions

## Installation

Copy to your Claude Code skills directory:

```bash
cp -r browser-automation ~/.claude/skills/
```

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

## Files

- `SKILL.md` - Quick reference (main skill file)
- `references/interactive-flow.md` - Login/auth handling
- `references/examples.md` - Code examples
- `references/troubleshooting.md` - Error handling & stuck detection

## Version

1.3.1

## License

MIT
