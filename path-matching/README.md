# Path Matching Demo

## What is Path Matching?

Path matching ensures that file paths are **identical** inside and outside the sandbox. This means:

- `/Users/ajeetsraina/project/src/app.js` on your host
- `/Users/ajeetsraina/project/src/app.js` inside the sandbox

**Same path. No translation needed.**

---

## Why Path Matching Matters

| Without Path Matching | With Path Matching (Docker Sandboxes) |
|----------------------|---------------------------------------|
| Host: `/Users/ajeet/project/src/Button.tsx` | Host: `/Users/ajeet/project/src/Button.tsx` |
| Container: `/workspace/src/Button.tsx` | Container: `/Users/ajeet/project/src/Button.tsx` ✅ |
| Error messages show `/workspace/...` — confusing! | Error messages show real paths |
| Copy-paste paths don't work | Copy-paste paths work |
| Mental translation required | Zero cognitive overhead |

---

## Demo: Proving Path Matching Works

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

---

### Step 2: Start the Sandbox

```bash
cd ~/meetup-jan/sandbox-testing
docker sandbox run claude
```

---

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

---

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

---

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

---

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

---

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

---

## Visual Comparison

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

---

## Real-World Benefits

### 1. Error Messages Make Sense

**Without path matching:**
```
Error: Cannot find module '/workspace/src/components/Button'
```
*Where is /workspace? 🤔*

**With path matching:**
```
Error: Cannot find module '/Users/ajeetsraina/meetup-jan/sandbox-testing/src/components/Button'
```
*I know exactly where that is! ✅*

---

### 2. Copy-Paste Paths Work

When Claude gives you a file path in an error message, you can copy it directly and use it on your host:

```bash
# Claude says: "Error in /Users/ajeetsraina/project/src/api/handler.ts"
# You can directly run on host:
code /Users/ajeetsraina/project/src/api/handler.ts
```

---

### 3. Git Paths Match

```bash
# Inside sandbox
git status
```

```
modified:   /Users/ajeetsraina/project/src/app.js
```

The path in git matches your actual filesystem — no confusion!

---

### 4. IDE Integration Works

If you have your IDE open on the host, file paths from sandbox errors can be clicked directly because they're real paths on your system.

---

## Test Summary

| Test | Result |
|------|--------|
| Full path access from sandbox | ✅ Working |
| Working directory matches host | ✅ Working |
| Relative paths work | ✅ Working |
| Files created in sandbox appear on host | ✅ Working |
| Files created on host appear in sandbox | ✅ Working |

---

## Key Takeaway

> **Path matching eliminates the cognitive overhead of translating between "container paths" and "real paths."**

With Docker Sandboxes, there's only ONE path — and it works everywhere.
