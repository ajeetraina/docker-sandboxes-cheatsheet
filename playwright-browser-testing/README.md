## Playwright Browser Testing

This demonstrates a practical use case: running browser tests in the sandbox without polluting your host system.

### Step 1: Create Project Directory

```bash
# On host
mkdir -p ~/meetup-jan/playwright-demo
cd ~/meetup-jan/playwright-demo

# Remove any existing sandbox
docker sandbox ls
docker sandbox rm <sandbox-id-if-exists>

# Start fresh sandbox
docker sandbox run claude
```

### Step 2: Create and Run Playwright Test

Inside the sandbox, ask Claude to create a Playwright test:

```
Create a simple Playwright test that:
1. Goes to https://collabnix.com
2. Takes a screenshot
3. Verifies the page title contains "Collabnix"
```

Claude will:
- Initialize the project with `npm init`
- Install Playwright and browsers (~500MB)
- Create the test file
- Run the test

**Result:**

```
● Bash(npx playwright test) timeout: 1m 0s
  ⎿  Running 1 test using 1 worker
       ✓  1 tests/collabnix.spec.ts:3:5 › Collabnix homepage (5.2s)

● Bash(ls -la collabnix-screenshot.png)
  ⎿  -rw-r--r-- 1 agent agent 666902 Jan 15 18:03 collabnix-screenshot.png

● Done. The Playwright test has been created and executed successfully.

  playwright-demo/
  ├── package.json
  ├── playwright.config.ts
  ├── tests/
  │   └── collabnix.spec.ts
  └── collabnix-screenshot.png  (generated)

  Result: 1 test passed in 6.3s
```
