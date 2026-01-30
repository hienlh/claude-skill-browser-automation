# Browser Automation Troubleshooting

## Common Errors

### Element Not Found

```javascript
// Bad - no wait
await page.click('#button');

// Good - wait for element
await page.waitForSelector('#button', { timeout: 5000 });
await page.click('#button');
```

### Navigation Timeout

```javascript
// Bad - default timeout may be too short
await page.goto('http://slow-site.com');

// Good - explicit timeout
await page.goto('http://slow-site.com', { timeout: 30000 });
```

### Selector Not Unique

```javascript
// Bad - multiple matches
await page.click('button');

// Good - specific selector
await page.click('button[data-testid="submit"]');
await page.click('button:has-text("Submit")');
await page.click('form#login button[type="submit"]');
```

## Retry Pattern

```javascript
mcp__playwright__browser_run_code({
  code: `async (page) => {
    const maxRetries = 3;
    for (let i = 0; i < maxRetries; i++) {
      try {
        await page.goto('http://localhost:3001');
        await page.waitForSelector('#app', { timeout: 5000 });
        return { success: true };
      } catch (e) {
        if (i === maxRetries - 1) throw e;
        await page.waitForTimeout(1000);
      }
    }
  }`
})
```

## Auto-Reload on Slow Loading

When page takes too long to load, reload and retry:

```javascript
mcp__playwright__browser_run_code({
  code: `async (page) => {
    const url = 'http://localhost:3020';
    const loadTimeout = 10000; // 10s
    const maxReloads = 3;

    for (let attempt = 0; attempt < maxReloads; attempt++) {
      try {
        await page.goto(url, { timeout: loadTimeout });
        // Wait for app to be ready (adjust selector)
        await page.waitForSelector('#app, #root, main', { timeout: 5000 });
        return { success: true, attempts: attempt + 1 };
      } catch (e) {
        if (attempt < maxReloads - 1) {
          console.log('Loading slow, reloading... attempt', attempt + 2);
          await page.reload({ timeout: loadTimeout });
        } else {
          throw new Error('Page failed to load after ' + maxReloads + ' attempts: ' + e.message);
        }
      }
    }
  }`
})
```

### Reload with Network Idle Check

```javascript
mcp__playwright__browser_run_code({
  code: `async (page) => {
    const url = 'http://localhost:3020';

    // Navigate with networkidle - waits for no network activity for 500ms
    try {
      await page.goto(url, {
        timeout: 15000,
        waitUntil: 'networkidle'
      });
    } catch (e) {
      // Timeout - try reload
      console.log('Initial load timeout, reloading...');
      await page.reload({ waitUntil: 'networkidle', timeout: 15000 });
    }

    // Verify content loaded
    const hasContent = await page.$('#app, #root, main, body > div');
    if (!hasContent) {
      await page.reload();
      await page.waitForSelector('#app, #root, main', { timeout: 10000 });
    }

    return { success: true, url: page.url() };
  }`
})
```

### Smart Loading Detection

```javascript
// Detect if page is still loading (spinner, skeleton, etc.)
async (page) => {
  const isLoading = async () => {
    return await page.evaluate(() => {
      // Check common loading indicators
      const spinners = document.querySelectorAll(
        '.spinner, .loading, [data-loading], .skeleton, .MuiCircularProgress-root'
      );
      return spinners.length > 0;
    });
  };

  // Wait for loading to complete, reload if stuck
  let waited = 0;
  const maxWait = 15000;
  const reloadThreshold = 10000;

  while (await isLoading()) {
    await page.waitForTimeout(500);
    waited += 500;

    if (waited >= reloadThreshold) {
      console.log('Loading stuck, reloading page...');
      await page.reload();
      waited = 0;
    }

    if (waited >= maxWait) {
      throw new Error('Page stuck in loading state');
    }
  }

  return { success: true };
}
```

## Stuck/Unresponsive UI Detection

### UI Frozen - Can't Click Anything

```javascript
mcp__playwright__browser_run_code({
  code: `async (page) => {
    const clickWithRetry = async (selector, maxRetries = 3) => {
      for (let i = 0; i < maxRetries; i++) {
        try {
          // Check if element is clickable
          const el = await page.waitForSelector(selector, { timeout: 5000 });
          await el.click({ timeout: 3000 });
          return true;
        } catch (e) {
          console.log('Click failed attempt', i + 1, e.message);

          // Check if page is frozen (overlay, modal blocking, etc.)
          const hasBlocker = await page.evaluate(() => {
            const blockers = document.querySelectorAll(
              '.modal-backdrop, .overlay, [class*="loading-overlay"], .MuiBackdrop-root'
            );
            return blockers.length > 0;
          });

          if (hasBlocker) {
            // Try to close/dismiss blocker
            await page.keyboard.press('Escape');
            await page.waitForTimeout(500);
          }

          if (i === maxRetries - 1) {
            // Last resort: reload
            console.log('UI unresponsive, reloading...');
            await page.reload();
            await page.waitForSelector(selector, { timeout: 10000 });
          }
        }
      }
      return false;
    };

    return await clickWithRetry('button[type="submit"]');
  }`
})
```

### Internal Data Loading (API calls pending)

```javascript
mcp__playwright__browser_run_code({
  code: `async (page) => {
    // Wait for internal loading to complete
    const waitForInternalLoad = async (timeout = 15000) => {
      const startTime = Date.now();

      while (Date.now() - startTime < timeout) {
        const isLoading = await page.evaluate(() => {
          // Check various loading indicators
          const indicators = [
            '.spinner', '.loading', '[data-loading="true"]',
            '.skeleton', '.MuiCircularProgress-root',
            '.ant-spin', '.el-loading', '[class*="loading"]',
            '.chakra-spinner', '.lds-ring'
          ];
          return indicators.some(sel => document.querySelector(sel));
        });

        if (!isLoading) return true;
        await page.waitForTimeout(500);
      }

      // Timeout - reload page
      console.log('Internal loading stuck, reloading...');
      await page.reload();
      return false;
    };

    await waitForInternalLoad();
  }`
})
```

### Action Without Response (Click but nothing happens)

```javascript
mcp__playwright__browser_run_code({
  code: `async (page) => {
    const actionWithVerify = async (action, verifyFn, maxRetries = 3) => {
      for (let i = 0; i < maxRetries; i++) {
        const beforeState = await page.url();

        // Perform action
        await action();
        await page.waitForTimeout(1000);

        // Verify action had effect
        const verified = await verifyFn();
        if (verified) return { success: true, attempts: i + 1 };

        console.log('Action had no effect, attempt', i + 1);

        // Check if we need to reload
        if (i < maxRetries - 1) {
          // Try action again, or reload if stuck
          const currentUrl = await page.url();
          if (currentUrl === beforeState) {
            console.log('Page stuck, reloading...');
            await page.reload();
            await page.waitForLoadState('networkidle');
          }
        }
      }
      return { success: false, error: 'Action had no effect after retries' };
    };

    // Example: Submit form and verify navigation
    return await actionWithVerify(
      async () => await page.click('button[type="submit"]'),
      async () => {
        const url = page.url();
        return !url.includes('/login'); // Verify we left login page
      }
    );
  }`
})
```

### Comprehensive Stuck Detection

```javascript
mcp__playwright__browser_run_code({
  code: `async (page) => {
    const isPageStuck = async () => {
      return await page.evaluate(() => {
        // 1. Check loading indicators
        const loadingSelectors = [
          '.spinner', '.loading', '[data-loading]', '.skeleton',
          '.MuiCircularProgress-root', '.ant-spin', '[class*="loading"]'
        ];
        const hasLoading = loadingSelectors.some(s => document.querySelector(s));

        // 2. Check blocking overlays
        const blockerSelectors = [
          '.modal-backdrop', '.overlay', '[class*="overlay"]',
          '.MuiBackdrop-root', '.ant-modal-mask'
        ];
        const hasBlocker = blockerSelectors.some(s => {
          const el = document.querySelector(s);
          return el && getComputedStyle(el).display !== 'none';
        });

        // 3. Check if body has overflow hidden (modal open)
        const bodyLocked = document.body.style.overflow === 'hidden';

        return { hasLoading, hasBlocker, bodyLocked };
      });
    };

    const handleStuck = async (maxWait = 10000) => {
      const startTime = Date.now();

      while (Date.now() - startTime < maxWait) {
        const status = await isPageStuck();

        if (!status.hasLoading && !status.hasBlocker) {
          return { success: true };
        }

        // Try to dismiss blockers
        if (status.hasBlocker || status.bodyLocked) {
          await page.keyboard.press('Escape');
          await page.waitForTimeout(300);
        }

        await page.waitForTimeout(500);
      }

      // Still stuck - reload
      console.log('Page stuck, reloading...');
      await page.reload();
      await page.waitForLoadState('domcontentloaded');
      return { success: true, reloaded: true };
    };

    return await handleStuck();
  }`
})
```

## Debug Mode

```javascript
// Add screenshot on failure
mcp__playwright__browser_run_code({
  code: `async (page) => {
    try {
      await page.goto('http://localhost:3001/login');
      await page.fill('#email', 'test@test.com');
      await page.click('#submit');
    } catch (e) {
      await page.screenshot({ path: 'error-screenshot.png' });
      throw e;
    }
  }`
})
```

## Wait Strategies

| Strategy | Use Case |
|----------|----------|
| `waitForSelector` | Wait for element to appear |
| `waitForURL` | Wait for navigation |
| `waitForLoadState('networkidle')` | Wait for all requests to finish |
| `waitForTimeout(ms)` | Hard wait (avoid if possible) |

## Common Selectors (Nxsys)

| Element | Selector |
|---------|----------|
| Login email | `input[name="email"]` |
| Login password | `input[name="password"]` |
| Submit button | `button[type="submit"]` |
| Data table row | `table tbody tr` |
| Modal confirm | `.modal-confirm button.confirm` |
| Toast success | `.toast-success`, `.Toastify__toast--success` |
