# 🌐 Browser Automation Skill

> **Save 97% context tokens** when doing browser automation with Claude Code!

Delegate browser tasks to subagents → Main context stays clean.

## ✨ Why Use This?

| Without Skill | With Skill |
|---------------|------------|
| ~3000 tokens per task | **~100 tokens** per task |
| Context bloats quickly | Context stays clean |
| Manual retry on failures | **Auto-reload** on stuck |
| No login handling | **Interactive flow** for auth |

### Key Benefits

- 🚀 **30x Context Reduction** - Only ~100 tokens vs ~3000 tokens per browser task
- 🔐 **Smart Auth Handling** - Pause for login, resume with credentials (NEEDS_INPUT protocol)
- 🔄 **Auto-Recovery** - Detects stuck pages, frozen UI, and auto-reloads
- ⚡ **Optimized Patterns** - Uses `browser_run_code` to batch multiple actions

## 📦 Installation

```bash
# Using skills CLI (recommended)
npx skills add hienlh/claude-skill-browser-automation

# Global install, no prompts
npx skills add hienlh/claude-skill-browser-automation -y -g
```

## 🚀 Quick Start

```javascript
Task({
  subagent_type: "general-purpose",
  description: "Browser: login and create user",
  prompt: `BROWSER TASK: Go to localhost:3000, login, create a new user
    Use Playwright MCP. Prefer browser_run_code.
    Return: SUCCESS/FAILED/NEEDS_INPUT + AgentId if pausing.`
})
```

## 🔐 Interactive Flow (Login/Auth)

When subagent encounters login:

```
1. Subagent → NEEDS_INPUT (fields: email, password, AgentId: abc123)
2. Main agent → Ask user for credentials
3. Resume → Task({ resume: "abc123", prompt: "credentials..." })
4. Subagent continues → SUCCESS
```

## 📁 Files

```
browser-automation/
├── SKILL.md                      # Quick reference
└── references/
    ├── interactive-flow.md       # Login/auth patterns
    ├── examples.md               # Code examples
    └── troubleshooting.md        # Error handling & auto-reload
```

## ⚙️ Requirements

### 1. Install Playwright MCP

```bash
# Add Playwright MCP to Claude Code
claude mcp add playwright -- npx @anthropic-ai/mcp-playwright@latest

# Verify installation
claude mcp list
```

Or add manually to `~/.claude.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-playwright@latest"]
    }
  }
}
```

### 2. Install Playwright Browsers

```bash
npx playwright install chromium
```

### 3. Task Tool Access

Ensure your Claude Code has access to the `Task` tool for subagent delegation.

## 📄 License

MIT
