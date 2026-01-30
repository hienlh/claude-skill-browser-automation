---
name: browser-automation
description: |
  Delegate browser automation tasks to a subagent to keep main context clean.
  Use this skill when you need to perform browser interactions (navigate, click, type, screenshot, etc.)
  without bloating the main agent's context window.

  Trigger phrases: "browser", "web page", "navigate to", "click on", "fill form", "screenshot", "test UI"
allowed-tools:
  - Task
  - mcp__playwright__*
  - AskUserQuestion
metadata:
  author: nxsys
  version: "1.3.0"
---

# Browser Automation Skill

Delegate browser tasks to subagent → main context stays clean (~100 tokens vs ~3000 tokens).

## Quick Template

```
Task({
  subagent_type: "general-purpose",
  description: "Browser: [description]",
  prompt: `BROWSER TASK: [what to do]
    Use Playwright MCP. Prefer browser_run_code.
    Return: SUCCESS/FAILED/NEEDS_INPUT + AgentId if pausing.`
})
```

## Interactive Flow (NEEDS_INPUT)

```
1. Subagent returns:    Status: NEEDS_INPUT | Fields: email, password | AgentId: abc123
2. Main agent:          AskUserQuestion for credentials
3. Resume subagent:     Task({ resume: "abc123", prompt: "email: x, password: y" })
```

## Key Tools

| Tool | Use |
|------|-----|
| `browser_run_code` | **PREFERRED** - Batch actions |
| `browser_navigate` | Go to URL |
| `browser_snapshot` | Page state (large output) |

## browser_run_code Pattern

```javascript
mcp__playwright__browser_run_code({
  code: `async (page) => {
    await page.goto('http://localhost:3001/login');
    await page.fill('#email', 'user@test.com');
    await page.click('button[type="submit"]');
    return { success: true, url: page.url() };
  }`
})
```

## Stuck Detection → Auto Reload

When page/UI stuck (loading, frozen, no response), reload:

```javascript
// 1. Internal loading stuck (spinners, skeletons)
const hasLoading = await page.$('.spinner, .loading, [class*="loading"]');
if (hasLoading) { await page.reload(); }

// 2. UI frozen (can't click) - try Escape first, then reload
await page.keyboard.press('Escape');
await page.waitForTimeout(500);
if (stillStuck) { await page.reload(); }

// 3. Action no response - verify result, retry or reload
await page.click('button');
await page.waitForTimeout(1000);
if (noChange) { await page.reload(); }
```
See `references/troubleshooting.md` for comprehensive patterns.

## Response Format

```
Status: SUCCESS/FAILED/NEEDS_INPUT
Action: [what was done]
AgentId: [for resume if NEEDS_INPUT]
Fields: [required fields if NEEDS_INPUT]
```

## References

- `references/interactive-flow.md` - Login/auth/user input handling
- `references/examples.md` - Detailed task examples
- `references/troubleshooting.md` - Error handling patterns
