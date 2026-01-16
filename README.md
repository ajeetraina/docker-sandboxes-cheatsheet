# Docker Sandboxes Cheatsheet and Tutorial

## Table of Contents

- [Getting Started](#getting-started)
  - [Create a Directory](#1-create-a-directory)
  - [Run the Sandbox](#2-run-the-sandbox)
  - [List and Inspect Sandboxes](#3-list-and-inspect-sandboxes)
  - [Managing Sandboxes](#4-managing-sandboxes)
- [Verify the Isolation](#verify-the-isolation)
  - [Test 1: Check if SSH Directory Exists](#test-1-check-if-ssh-directory-exists)
  - [Test 2: Try to Access AWS Credentials](#test-2-try-to-access-aws-credentials)
  - [Test 3: Try to Access Documents Folder](#test-3-try-to-access-documents-folder)
  - [Test 4: Confirm Project Folder Access](#test-4-confirm-project-folder-access)
  - [Test 5: The Nuclear Option](#test-5-the-nuclear-option)
  - [Test 5b: Sandbox Home Directory Contents](#test-5b-sandbox-home-directory-contents)
  - [Test 6: Verify Git Identity Injection](#test-6-verify-git-identity-injection)
- [Test Path Matching](#test-7-path-matching)
- [Test State Persistence](#test-8-state-persistence)
  - [Step 1: Install a Package](#step-1-install-a-package)
  - [Step 2: Exit the Sandbox](#step-2-exit-the-sandbox)
  - [Step 3: Re-enter and Verify](#step-3-re-enter-and-verify)
- [Test Environment Variables](#test-9-environment-variables)
- [Test Docker Socket Access](#test-10-docker-socket-access)
- [Real-World Demo: Playwright Browser Testing](#test-11-real-world-demo---playwright-browser-testing)
- [Test Summary](#test-summary)
- [Key Takeaways](#key-takeaways)

---

## Getting Started

### Prerequisites

- Docker Desktop 4.56+

### 1. Create a Directory

```bash
mkdir -p /Users/ajeetsraina/sandbox-testing
cd /Users/ajeetsraina/sandbox-testing
```

### 2. Run the Sandbox

```bash
docker sandbox run
```

```
docker: 'docker sandbox run' requires at least 1 argument

Usage:  docker sandbox run [options] <agent> [agent-options]

See 'docker sandbox run --help' for more information

Available Agents:
  claude          Run Claude AI agent inside a sandbox
  gemini          Run Gemini AI agent inside a sandbox
```

```bash
docker sandbox run claude
```

### 3. List and Inspect Sandboxes

```bash
docker sandbox ls
```

```
SANDBOX ID     TEMPLATE                               NAME                               WORKSPACE                            STATUS    CREATED
275d94b417bf   docker/sandbox-templates:claude-code   claude-sandbox-2026-01-11-004116   /Users/ajeetsraina/sandbox-testing   running   2026-01-10 19:12:10
```

```bash
docker sandbox inspect 275d94b417bf
```

```json
[
  {
    "id": "275d94b417bf8f4c29f6f3c7317f20f6b9636b3f3121d303149a066d8330428e",
    "name": "claude-sandbox-2026-01-11-004116",
    "workspace": "/Users/ajeetsraina/sandbox-testing",
    "created_at": "2026-01-10T19:12:10.888151834Z",
    "status": "running",
    "template": "docker/sandbox-templates:claude-code",
    "labels": {
      "com.docker.sandbox.agent": "claude",
      "com.docker.sandbox.credentials": "sandbox",
      "com.docker.sandbox.workingDirectory": "/Users/ajeetsraina/sandbox-testing",
      "com.docker.sandbox.workingDirectoryInode": "186434127",
      "com.docker.sandboxes": "templates",
      "com.docker.sandboxes.base": "ubuntu:questing",
      "com.docker.sandboxes.flavor": "claude-code",
      "com.docker.sdk": "true",
      "com.docker.sdk.client": "0.1.0-alpha011",
      "com.docker.sdk.container": "0.1.0-alpha012",
      "com.docker.sdk.lang": "go",
      "docker/sandbox": "true",
      "org.opencontainers.image.ref.name": "ubuntu",
      "org.opencontainers.image.version": "25.10"
    }
  }
]
```

> **Note:** The `docker/sandbox-templates:claude-code` image includes Claude Code with automatic credential management, plus development tools (Docker CLI, GitHub CLI, Node.js, Go, Python 3, Git, ripgrep, jq). It runs as a non-root agent user with sudo access and launches Claude with `--dangerously-skip-permissions` by default.

### 4. Managing Sandboxes

Since Docker enforces one sandbox per workspace, the same sandbox is reused each time you run `docker sandbox run <agent>` in a given directory. To create a fresh sandbox, you need to remove the existing one first:

```bash
docker sandbox ls           # Find the sandbox ID
docker sandbox rm <sandbox-id>
docker sandbox run <agent>  # Creates a new sandbox
```

---

## Verify the Isolation

### Test 1: Check if SSH Directory Exists

```bash
ls -la ~/.ssh/
```

**Result:**

```
Bash(ls -la ~/.ssh/)
  ⎿  Error: Exit code 2
     ls: cannot access '/home/agent/.ssh/': No such file or directory
```

✅ **That's the sandbox working!**

Notice the path: `/home/agent/.ssh/` — the sandbox can't see your host's SSH keys at all. They simply **don't exist** inside the container.

---

### Test 2: Try to Access AWS Credentials

```bash
ls -la ~/.aws/
```

**Result:**

```
Bash(ls -la ~/.aws/)
  ⎿  Error: Exit code 2
     ls: cannot access '/home/agent/.aws/': No such file or directory
```

✅ **AWS credentials protected!**

---

### Test 3: Try to Access Documents Folder

```bash
ls ~/Documents/
```

**Result:**

```
● The ~/Documents/ directory doesn't exist in this environment.

  This is a sandboxed environment with a minimal filesystem. The current working directory is
  /Users/ajeetsraina/meetup-jan/sandbox-testing.
  
  Would you like me to list what's in the current working directory or the home directory instead?
```

✅ **Documents folder blocked!**

The sandbox is "secure by default" — you don't have to think about what to exclude. It only sees your project folder.

---

### Test 4: Confirm Project Folder Access

```bash
ls -la /Users/ajeetsraina/meetup-jan/sandbox-testing/
```

**Result:**

```
Bash(ls -la /Users/ajeetsraina/meetup-jan/sandbox-testing/)
  ⎿  total 93416
     drwxr-xr-x 8 agent agent      256 Jan 15 15:09 .
     drwxr-xr-x 3 root  root      4096 Jan 15 15:52 ..
     … +7 lines (ctrl+o to expand)
```

✅ **It works!**

The sandbox **CAN** see your project directory at the exact same path as your host machine. That's **path matching** in action.

---

### Test 5: The Nuclear Option

Can it delete your home directory?

```bash
rm -rf ~/
```

**Result:**

```
I won't run that command. rm -rf ~/ would recursively delete the entire home directory, which is destructive
and irreversible.

If you have a legitimate need to clean up specific files or directories, let me know what you're actually
trying to accomplish and I can help with a targeted approach.
```

Claude's safety guardrails kicked in! But even if it ran, see Test 5b for why it wouldn't matter.

---

### Test 5b: Sandbox Home Directory Contents

```bash
ls -la ~/
```

**Result:**

```
Bash(ls -la ~/)
  ⎿  total 64
     drwxr-xr-x 1 agent agent 4096 Jan 15 16:23 .
     drwxr-xr-x 1 root  root  4096 Jan 10 00:23 ..
     … +9 lines (ctrl+o to expand)

● The home directory contains:
  ┌──────────────┬──────┬─────────────────────┐
  │     Item     │ Type │     Description     │
  ├──────────────┼──────┼─────────────────────┤
  │ .cache/      │ dir  │ Cache files         │
  ├──────────────┼──────┼─────────────────────┤
  │ .claude/     │ dir  │ Claude Code config  │
  ├──────────────┼──────┼─────────────────────┤
  │ .claude.json │ file │ Claude settings     │
  ├──────────────┼──────┼─────────────────────┤
  │ .docker/     │ dir  │ Docker config       │
  ├──────────────┼──────┼─────────────────────┤
  │ .gitconfig   │ file │ Git configuration   │
  ├──────────────┼──────┼─────────────────────┤
  │ .local/      │ dir  │ Local user data     │
  ├──────────────┼──────┼─────────────────────┤
  │ .npm/        │ dir  │ npm cache           │
  ├──────────────┼──────┼─────────────────────┤
  │ workspace/   │ dir  │ Workspace directory │
  └──────────────┴──────┴─────────────────────┘
```

🎯 **This is the key insight!**

The sandbox's home (`/home/agent/`) is **completely separate** from your real home (`/Users/ajeetsraina/`).

| If `rm -rf ~/` runs... | Without Sandbox | With Sandbox |
|------------------------|-----------------|--------------|
| What gets deleted | Your **real** home: Documents, SSH keys, years of work | Just `/home/agent/`: cache, npm, empty sandbox stuff |
| Impact | **Catastrophic** 💀 | **Nothing important** ✅ |

---

### Test 6: Verify Git Identity Injection

```bash
cat ~/.gitconfig
```

**Result:**

```
Read(~/.gitconfig)
  ⎿  Read 3 lines

● [safe]
      directory = /Users/ajeetsraina/meetup-jan/sandbox-testing

  The gitconfig just has a safe.directory setting, which tells Git to trust the 
  /Users/ajeetsraina/meetup-jan/sandbox-testing directory.
```

**Test 6b: Check full Git config**

```bash
git config --global --list
```

**Result:**

```
safe.directory=/Users/ajeetsraina/meetup-jan/sandbox-testing
```

⚠️ **Note:** Git identity (`user.name`, `user.email`) was not auto-injected despite being configured on the host. Will work with Sandboxes team and get it fixed.

---

## Test 7: Path Matching

Path matching ensures that file paths are **identical** inside and outside the sandbox. This is crucial for:

- Error messages that make sense
- Copy-paste paths that work
- IDE integration
- Git path consistency

### Why Path Matching Matters

| Without Path Matching | With Path Matching (Docker Sandboxes) |
|----------------------|---------------------------------------|
| Host: `/Users/ajeet/project/src/Button.tsx` | Host: `/Users/ajeet/project/src/Button.tsx` |
| Container: `/workspace/src/Button.tsx` | Container: `/Users/ajeet/project/src/Button.tsx` ✅ |
| Error messages show `/workspace/...` — confusing! | Error messages show real paths |
| Copy-paste paths don't work | Copy-paste paths work |

### Step 1: Create a File on HOST

```bash
# On your host terminal
mkdir -p ~/meetup-jan/sandbox-testing/src/components
echo "export const Button = () => <button>Click me</button>" > ~/meetup-jan/sandbox-testing/src/components/Button.tsx
```

Verify it exists:

```bash
cat ~/meetup-jan/sandbox-testing/src/components/Button.tsx
```

**Result:**

```
export const Button = () => <button>Click me</button>
```

### Step 2: Start the Sandbox

```bash
cd ~/meetup-jan/sandbox-testing
docker sandbox run claude
```

### Step 3: Access File Using FULL PATH Inside Sandbox

Inside the sandbox, use the exact same path as your host:

```bash
cat /Users/ajeetsraina/meetup-jan/sandbox-testing/src/components/Button.tsx
```

**Result:**

```
● Bash(cat /Users/ajeetsraina/meetup-jan/sandbox-testing/src/components/Button.tsx)
  ⎿  export const Button = () => <button>Click me</button>
```

✅ **Same path works inside the sandbox!**

### Step 4: Verify Working Directory

```bash
pwd
```

**Result:**

```
● Bash(pwd)
  ⎿  /Users/ajeetsraina/meetup-jan/sandbox-testing
```

✅ **Working directory matches your host path!**

### Step 5: Access with Relative Path

```bash
cat src/components/Button.tsx
```

**Result:**

```
● Bash(cat src/components/Button.tsx)
  ⎿  export const Button = () => <button>Click me</button>
```

✅ **Relative paths work too!**

### Step 6: Create a File INSIDE Sandbox

Create a new file using the full path:

```bash
echo "console.log('created inside sandbox')" > /Users/ajeetsraina/meetup-jan/sandbox-testing/src/utils.js
```

Verify inside sandbox:

```bash
cat /Users/ajeetsraina/meetup-jan/sandbox-testing/src/utils.js
```

**Result:**

```
● Bash(cat /Users/ajeetsraina/meetup-jan/sandbox-testing/src/utils.js)
  ⎿  console.log('created inside sandbox')
```

### Step 7: Verify File Exists on HOST

Exit the sandbox:

```bash
exit
```

Check on your host:

```bash
cat ~/meetup-jan/sandbox-testing/src/utils.js
```

**Result:**

```
console.log('created inside sandbox')
```

✅ **File created inside sandbox appears on host at the same path!**

### Visual Comparison

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    REGULAR DOCKER CONTAINER                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  HOST                              CONTAINER                            │
│  /Users/ajeet/project/             /workspace/                          │
│  ├── src/                          ├── src/                             │
│  │   └── app.js                    │   └── app.js                       │
│  └── package.json                  └── package.json                     │
│                                                                         │
│  ❌ Paths are DIFFERENT                                                 │
│  ❌ Error: "File not found at /workspace/src/app.js"                    │
│  ❌ You think: "Where is /workspace? That's not my path!"               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                      DOCKER SANDBOXES                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  HOST                              SANDBOX                              │
│  /Users/ajeet/project/             /Users/ajeet/project/                │
│  ├── src/                          ├── src/                             │
│  │   └── app.js                    │   └── app.js                       │
│  └── package.json                  └── package.json                     │
│                                                                         │
│  ✅ Paths are IDENTICAL                                                 │
│  ✅ Error: "File not found at /Users/ajeet/project/src/app.js"          │
│  ✅ You think: "I know exactly where that is!"                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Path Matching Summary

| Test | Result |
|------|--------|
| Full path access from sandbox | ✅ Working |
| Working directory matches host | ✅ Working |
| Relative paths work | ✅ Working |
| Files created in sandbox appear on host | ✅ Working |
| Files created on host appear in sandbox | ✅ Working |

---

## Test 8: State Persistence

### Step 1: Install a Package

```bash
npm install -g cowsay
```

Then test it works:

```bash
cowsay "Hello from sandbox"
```

**Result:**

```
● Bash(cowsay "hello from sandbox")
  ⎿   ____________________
     < hello from sandbox >
      --------------------
            \   ^__^
             \  (oo)\_______
                (__)\       )\/\
                    ||----w |
                    ||     ||
```

### Step 2: Exit the Sandbox

```bash
exit
```

Or type `/exit` in Claude Code.

### Step 3: Re-enter and Verify

```bash
docker sandbox run claude
```

Then test if cowsay is still there:

```bash
cowsay "I persisted!"
```

**Result:**

```
● Done! The cow has spoken.
```

✅ **State persistence confirmed!**

Unlike a regular `docker run` (which loses everything on exit), Docker Sandbox **remembered** the installed package.

---

## Test 9: Environment Variables

Environment variables must be set at sandbox creation time.

### Step 1: Remove Existing Sandbox

```bash
# On your host terminal
docker sandbox ls
docker sandbox rm <sandbox-id>
```

### Step 2: Create Sandbox with Environment Variables

```bash
docker sandbox run -e MY_SECRET=supersecret123 -e APP_ENV=development claude
```

### Step 3: Verify Inside Sandbox

```bash
echo $MY_SECRET
echo $APP_ENV
```

**Result:**

```
● Bash(echo $MY_SECRET)
  ⎿  supersecret123

● Bash(echo $APP_ENV)
  ⎿  development
```

### Step 4: Confirm Full Environment Access

```bash
printenv | grep -E "MY_SECRET|APP_ENV"
```

**Result:**

```
● Bash(printenv | grep -E "MY_SECRET|APP_ENV")
  ⎿  MY_SECRET=supersecret123
     APP_ENV=development
```

✅ **Environment variables working!**

> ⚠️ **Important Limitation:** You cannot hot-reload environment variables. To change them, you must remove and recreate the sandbox (which loses installed packages).

---

## Test 10: Docker Socket Access

This allows the agent to run Docker commands inside the sandbox.

> ⚠️ **Security Warning:** Mounting the Docker socket grants the agent full access to your Docker daemon, which has root-level privileges. Only use this when necessary.

### Step 1: Remove Existing Sandbox

```bash
# On your host terminal
exit
docker sandbox rm <sandbox-id>
```

### Step 2: Create Sandbox with Docker Socket

```bash
docker sandbox run --mount-docker-socket claude
```

### Step 3: Test Docker Access

```bash
docker ps
```

**Result:**

```
● Bash(docker ps)
  ⎿  Error: Exit code 1
     permission denied while trying to connect to the docker API at unix:///var/run/docker.sock
```

Docker socket requires `sudo` inside the sandbox:

```bash
sudo docker ps
```

**Result:**

```
● Bash(sudo docker ps)
  ⎿  CONTAINER ID   IMAGE                                  COMMAND                  CREATED              STATUS
     dbab95b2ae42   docker/sandbox-templates:claude-code   "sh -c 'sleep 5; if …"   About a minute ago   Up About a minute
     … +9 lines
```

✅ **Docker socket access working!**

The agent can now:
- List and manage containers
- Build Docker images
- Run `docker compose` commands
- Execute integration tests with Testcontainers

---

## Test 11: Real-World Demo - Playwright Browser Testing

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

### Step 3: Verify Isolation on Host

Exit the sandbox and check your host:

```bash
exit
```

**Check what's on your host:**

```bash
# Screenshot IS in your project (shared via mount) ✅
ls -la ~/meetup-jan/playwright-demo/collabnix-screenshot.png

# Playwright browsers are NOT on your host ✅
ls ~/.cache/ms-playwright/
```

**Result:**

| Location | On Host? | Why? |
|----------|----------|------|
| `collabnix-screenshot.png` | ✅ Yes | Project folder is mounted |
| `node_modules/` | ✅ Yes | Project folder is mounted |
| `~/.cache/ms-playwright/` (500MB browsers) | ❌ No | Isolated in sandbox |
| `~/.npm/` cache | ❌ No | Isolated in sandbox |

✅ **This is the power of Docker Sandboxes!**

- Your project files are accessible and shared
- Heavy dependencies (browsers, caches) stay in the sandbox
- Your host system stays clean
- Re-enter the sandbox later and Playwright is still installed

---

## Test Summary

| Feature | Expected | Result |
|---------|----------|--------|
| 🔒 SSH keys blocked | Blocked | ✅ Working |
| 🔒 AWS credentials blocked | Blocked | ✅ Working |
| 🔒 Documents blocked | Blocked | ✅ Working |
| 📁 Project folder accessible | Accessible | ✅ Working |
| 🎯 Path matching | Same paths | ✅ Working |
| 💾 State persistence | Persists | ✅ Working |
| 🔧 Environment variables | Available | ✅ Working |
| 🐳 Docker socket access | With sudo | ✅ Working |
| 🎭 Playwright isolation | Browsers isolated | ✅ Working |
| 🪪 Git identity injection | Auto-injected | ⚠️ Not working |

---

## Key Takeaways

| Regular Container | Docker Sandbox |
|-------------------|----------------|
| You manually decide what to mount | Auto-mounts **only** project directory |
| Could accidentally mount `~/.ssh`, `~/.aws` | **Automatically excludes** sensitive dirs |
| Different paths inside vs outside | **Same paths** (path matching) |
| No Git identity | Should auto-inject Git config |
| State lost on exit | **State persists** per workspace |

**Docker Sandboxes = Secure by Default** 🛡️
