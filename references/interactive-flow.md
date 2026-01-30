# Interactive Browser Flow (User Input)

## Overview

When a subagent encounters a situation requiring user input (login, 2FA, captcha, confirmation), it should:

1. **Pause** - Return `NEEDS_INPUT` status with required fields
2. **Main agent asks** - Uses `AskUserQuestion` tool
3. **Resume** - Main agent resumes subagent with user's input

## Complete Example: Login Flow

### Initial Task (Main Agent)

```javascript
const result = await Task({
  subagent_type: "general-purpose",
  description: "Browser: access dashboard",
  prompt: `BROWSER TASK: Navigate to http://localhost:3001/dashboard

    Use Playwright MCP. If login required, return NEEDS_INPUT with fields.
    Return: Status (SUCCESS/FAILED/NEEDS_INPUT) + AgentId if pausing.`
});
```

### Subagent Detects Login (Subagent Response)

```javascript
// Subagent code
mcp__playwright__browser_run_code({
  code: `async (page) => {
    await page.goto('http://localhost:3001/dashboard');

    // Check if redirected to login
    if (page.url().includes('/login')) {
      // Detect form fields
      const fields = await page.$$eval('input', inputs =>
        inputs.map(i => i.name || i.id).filter(Boolean)
      );
      return {
        status: 'NEEDS_INPUT',
        reason: 'Login required',
        fields: fields, // ['email', 'password']
        url: page.url()
      };
    }

    return { status: 'SUCCESS', url: page.url() };
  }`
})

// Subagent returns:
// Status: NEEDS_INPUT
// Reason: Login required
// Fields: email, password
// URL: http://localhost:3001/login
// AgentId: abc123
```

### Main Agent Asks User

```javascript
// Main agent parses NEEDS_INPUT and asks user
AskUserQuestion({
  questions: [
    {
      question: "Login required for http://localhost:3001/login. Email?",
      header: "Email",
      options: [
        { label: "test@test.com", description: "Default test account" },
        { label: "Enter custom", description: "Provide different email" }
      ]
    },
    {
      question: "Password?",
      header: "Password",
      options: [
        { label: "Use default", description: "password123" },
        { label: "Enter custom", description: "Provide different password" }
      ]
    }
  ]
});
```

### Resume Subagent with Credentials

```javascript
const finalResult = await Task({
  resume: "abc123",  // AgentId from subagent
  prompt: `USER PROVIDED CREDENTIALS:
    email: test@test.com
    password: password123

    Continue login and access dashboard.
    Return: Status (SUCCESS/FAILED) + result.`
});
```

### Subagent Continues

```javascript
// Resumed subagent receives credentials and continues
mcp__playwright__browser_run_code({
  code: `async (page) => {
    // Page is still on login form (preserved state)
    await page.fill('input[name="email"]', 'test@test.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button[type="submit"]');
    await page.waitForURL('**/dashboard');
    return { status: 'SUCCESS', url: page.url() };
  }`
})

// Final response:
// Status: SUCCESS
// Action: Logged in and accessed dashboard
// URL: http://localhost:3001/dashboard
```

## Other NEEDS_INPUT Scenarios

### 2FA / OTP

```
Status: NEEDS_INPUT
Reason: 2FA required
Fields: otp_code
Message: "Check your email/phone for 6-digit code"
AgentId: xyz789
```

### Captcha

```
Status: NEEDS_INPUT
Reason: Captcha detected
Fields: manual_solve
Message: "Please solve captcha manually, then confirm"
Screenshot: captcha.png
AgentId: xyz789
```

### Confirmation Dialog

```
Status: NEEDS_INPUT
Reason: Confirmation required
Fields: confirm
Message: "Delete 5 records? This cannot be undone."
Options: [yes, no]
AgentId: xyz789
```

## Field Detection Code Snippet

```javascript
// Detect input fields on any page
async (page) => {
  const fields = await page.$$eval('input, select, textarea', elements =>
    elements.map(el => ({
      name: el.name || el.id,
      type: el.type || el.tagName.toLowerCase(),
      label: el.labels?.[0]?.textContent || el.placeholder || ''
    })).filter(f => f.name)
  );
  return fields;
}
```

## Tips

1. **Always include AgentId** - Required for resume
2. **Preserve browser state** - Don't close browser when returning NEEDS_INPUT
3. **Be specific about fields** - List exactly what's needed
4. **Include context** - URL, screenshot path, error message if any
