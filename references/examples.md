# Browser Automation Examples

## Login Flow

```javascript
mcp__playwright__browser_run_code({
  code: `async (page) => {
    await page.goto('http://localhost:3001/login');
    await page.fill('input[name="email"]', 'test@test.com');
    await page.fill('input[name="password"]', 'password');
    await page.click('button[type="submit"]');
    await page.waitForURL('**/dashboard', { timeout: 10000 });
    return { success: true, url: page.url() };
  }`
})
```

## Create Timesheet (Nxsys)

```javascript
mcp__playwright__browser_run_code({
  code: `async (page) => {
    // Navigate to timesheet
    await page.goto('http://localhost:3001/timesheet/new');

    // Fill form
    await page.fill('[data-testid="rate"]', '150');
    await page.fill('[data-testid="units"]', '5');
    await page.selectOption('[data-testid="subcontractor"]', 'SR-26');

    // Submit
    await page.click('button[type="submit"]');
    await page.waitForSelector('.success-message');

    // Get result
    const total = await page.textContent('[data-testid="total"]');
    return { success: true, total };
  }`
})
```

## Extract Table Data

```javascript
mcp__playwright__browser_run_code({
  code: `async (page) => {
    await page.goto('http://localhost:3001/payroll');

    // Wait for table
    await page.waitForSelector('table tbody tr');

    // Extract rows
    const rows = await page.$$eval('table tbody tr', rows =>
      rows.map(row => ({
        id: row.querySelector('td:nth-child(1)')?.textContent,
        name: row.querySelector('td:nth-child(2)')?.textContent,
        amount: row.querySelector('td:nth-child(3)')?.textContent,
      }))
    );

    return { success: true, count: rows.length, rows };
  }`
})
```

## Take Screenshot with Wait

```javascript
mcp__playwright__browser_run_code({
  code: `async (page) => {
    await page.goto('http://localhost:3001/dashboard');
    await page.waitForLoadState('networkidle');
    await page.screenshot({ path: 'dashboard.png', fullPage: true });
    return { success: true, path: 'dashboard.png' };
  }`
})
```

## Handle Modal/Dialog

```javascript
mcp__playwright__browser_run_code({
  code: `async (page) => {
    await page.click('button.delete');

    // Wait for confirmation modal
    await page.waitForSelector('.modal-confirm');
    await page.click('.modal-confirm button.confirm');

    // Wait for success
    await page.waitForSelector('.toast-success');
    return { success: true, action: 'deleted' };
  }`
})
```
