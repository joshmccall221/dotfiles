# Josh's CLAUDE.md — Global Developer Context

> Place this file at `~/.claude/CLAUDE.md` on each machine for global effect,
> or at the root of any project repo for project-specific context.

---

## Who I Am

I'm a .NET / Azure developer (APIs, managed identity, OBO flow, Kusto/ADX)
with frontend work and Playwright integration testing. I use VS Code with
GitHub Copilot (Claude models). I work across multiple machines and want
Claude to act as a senior engineer who knows my setup — not a generic assistant.

**My working style:** I act as architect and reviewer. Claude works autonomously
on implementation details. I make decisions at forks.

---

## My Devices & Roles

| Device | Role | Claude Access |
|---|---|---|
| Mac mini (work) | Primary compute hub — always on | Enterprise (unlimited) |
| iMac | Secondary desktop — also on PiKVM | Enterprise (unlimited) |
| Windows laptop | Mobile dev machine — also on PiKVM | Personal + enterprise |
| iPhone / iPad | Monitor, steer, SSH from anywhere | claude.ai mobile |
| Apple Vision | Context review, planning, reading diffs | claude.ai |
| PiKVM | Out-of-band KVM access to all machines | n/a — infrastructure only |

**Key principle:** Never start a fresh session on Windows. SSH into Mac mini
and attach to an existing tmux/cmux session instead. This preserves context
and uses enterprise Claude.

---

## PiKVM — Role and Limitations

PiKVM provides browser-based KVM (keyboard/video/mouse) access to the iMac,
Mac mini, and Windows laptop. Understand what it's good for vs. not.

**Use PiKVM for:**
- Full GUI access to any machine from a browser when you need it
- Out-of-band recovery — machine crashed, hung, or lost network
- BIOS/boot access, system settings that require physical presence
- Rebooting a stuck machine remotely

**Do NOT use PiKVM terminal for daily workflow:**
- PiKVM's built-in terminal accesses the *PiKVM device itself*, not your machines
- KVM screen streaming is heavy — video feed, battery-expensive on mobile
- No native tmux integration — you're watching a screen, not a real session
- Laggy compared to direct SSH, especially on iPhone

**Mobile access hierarchy (fastest → heaviest):**
1. **Blink/Prompt → SSH → tmux** — for terminal work, agent monitoring, steering
2. **claude.ai mobile** — for planning, reviewing STATUS.md, light prompting
3. **PiKVM in browser** — for GUI access or machine recovery only

---

## How I Want Claude to Behave

### Before starting any non-trivial task:
1. **Assess complexity** — can this be done better sequentially, in parallel
   with subagents, or delegated to run overnight?
2. **Surface a better approach if one exists** — say so briefly before starting.
   Example: "This could run as parallel agents overnight vs. sequentially now —
   which do you want?"
3. **Give me 2-3 options with tradeoffs** for anything architectural.
4. **Wait for me to pick** before executing on approach decisions.

### While working:
- Work autonomously on implementation details — don't ask me about things
  you can reasonably decide.
- **Explain what you did and why** after significant decisions, not before.
- If you're uncertain about direction, **stop and ask** rather than running
  500 lines the wrong way.
- Write a `STATUS.md` when pausing on long tasks: what's done, what's next,
  what's blocked.

### When I'm away / overnight tasks:
- Produce concrete artifacts (files, PRs, STATUS.md) — not just terminal output.
- End condition should be something I can review from my phone.
- Prefer reversible actions. Don't delete or overwrite without confirmation.

---

## Workflow Decision Framework

When I give you a task, use this to recommend an approach:

```
IF task is complex AND I'm at my computer AND Claude quota available
  → Recommend: VSCode + Copilot (enterprise, interactive) or Claude Code

IF task is parallelizable (multiple features / test suites / research threads)
  → Recommend: Claude Code Agent Teams or subagents

IF I'm stepping away / running overnight AND Claude quota available
  → Recommend: Copilot CLI or Claude Code in cmux/tmux (detachable, resumable)

IF Claude quota exhausted BUT Copilot enterprise available
  → Recommend: Copilot CLI with Claude Opus/Sonnet model selected

IF both Claude and Copilot quota exhausted
  → Recommend: Ollama + local LLM for low-reasoning tasks

IF task is large enough to split across agents
  → Mention Ruflo / Agent Teams as an option, explain briefly
```

---

## GitHub Copilot CLI — Enterprise Setup

Copilot CLI is the terminal equivalent of VS Code Copilot. It uses the same
enterprise subscription, supports Claude Opus 4.6 / Sonnet 4.6 / Haiku 4.5,
and is detachable in tmux. Use it when stepping away from VS Code or when
Claude Code quota is low.

### Install (Mac mini)

```bash
# Requires Node.js 22+
npm install -g @github/copilot

# Or via Homebrew
brew install gh-copilot
```

### Authenticate (enterprise)

```bash
# Launch and follow browser OAuth prompt
copilot
/login

# Verify account
/user

# For enterprise with custom hostname, set env var first:
export GITHUB_ENTERPRISE_URL=https://your-org.ghe.com
```

> Your org admin must enable Copilot CLI in enterprise settings first.
> Check with your IT/GitHub admin if `/login` fails on enterprise account.

### Model Selection

```bash
# Default is Claude Sonnet 4.5 — switch mid-session:
/model

# Or launch with specific model:
copilot --model claude-opus-4-6

# Available Claude models (via /model picker):
# claude-opus-4-6     ← best quality, use for complex tasks
# claude-sonnet-4-6   ← default, good balance
# claude-haiku-4-5    ← fastest, cheap tasks
```

### Key Commands

```bash
copilot              # start interactive session
/plan                # plan a task before executing
/model               # switch model mid-session
/delegate            # hand off to background cloud agent (&prefix also works)
/fleet               # run task across multiple models in parallel
/ide                 # open current work in VS Code
/compact             # compress context when session gets long
/logout              # switch accounts
```

---

## Transitioning from VS Code → Command Line

Use this when you need to step away from your desk mid-task. The goal is
to hand off cleanly so work continues without you sitting there.

### Step 1 — Capture context before leaving VS Code

In VS Code Copilot chat, run:
```
Summarize exactly what we've done, what's in progress, what's next,
and any decisions made. Format as a STATUS.md I can hand to a CLI agent.
```

Copy the output or let it write `STATUS.md` to the project root.

### Step 2 — Open a named cmux/tmux session

```bash
# In cmux: Cmd+T for new tab, name it after the task
# Or in terminal:
tmux new -s taskname
cd /path/to/project
```

### Step 3 — Kick off Copilot CLI or Claude Code with context

```bash
# Option A: Copilot CLI (uses enterprise subscription)
copilot
# Then paste or reference STATUS.md:
"Read STATUS.md and continue the work described there. Ask if unclear."

# Option B: Claude Code
claude
"Read STATUS.md and continue. Produce updated STATUS.md when pausing."
```

### Step 4 — Detach and walk away

```bash
# cmux: just close the lid / switch apps — session persists
# tmux: Ctrl+b d
```

### Step 5 — Check from phone

```bash
# SSH in from Blink/Prompt on iPhone
ssh macmini -t "tmux attach -t taskname"
# or in cmux notification: tap the tab with the ring
```

### Step 6 — Return to VS Code when back at desk

```bash
# From Copilot CLI:
/ide
# This opens current state in VS Code

# From Claude Code: just open VS Code in same directory
# All file changes are already on disk
```

---

## My Stack (for context-aware suggestions)

- **Backend:** .NET APIs, Azure (managed identity, OBO flow), Kusto / ADX
- **Frontend:** Playwright for integration testing
- **Editor:** VS Code + GitHub Copilot (Claude)
- **Shell:** tmux, zsh (Mac), PowerShell / WSL (Windows)
- **Local LLM:** Ollama (Mac mini) — use as fallback when quota is low
- **Agent tools:** Claude Code, exploring Ruflo / Agent Teams
- **Version control:** Git (assume standard branching, PRs)

---

## Model Selection — Assess Before Starting

Before every task, evaluate complexity and recommend the right model.
State the recommendation briefly before starting — don't just proceed.

```
Haiku    → quick lookups, simple edits, boilerplate, one-liners
Sonnet   → most daily tasks, single-file changes, moderate reasoning
Opus     → complex multi-file refactors, hard bugs, architecture decisions,
           anything touching auth/security/Azure, overnight unattended tasks
```

**How to say it:**
- "This is a quick config change — Sonnet is fine here."
- "This touches OBO flow and spans 4 files — I'd recommend switching to Opus."
- "For an overnight agent run, Opus gives better reliability. Switch with
  `claude --model claude-opus-4-6` or `/model` in Copilot CLI."

**When to suggest Agent Teams or parallel:**
- Task has 3+ independent workstreams → suggest splitting
- Task will saturate context window → suggest subagents
- ETA > 30 min sequential → ask if overnight parallel is preferred

---

## Code Review — Understanding What Changed

I don't want to vibe code. After any non-trivial task, help me understand
what was built well enough to explain it to a colleague or debug it later.

### Always do before starting a task:
```bash
git checkout -b feature/task-name   # never work on main
```

### Always do after completing a task:

**Explain the changes:**
After finishing, provide a short post-task summary:
- What are the 3-5 most significant changes and why each was made?
- What patterns or decisions should I understand?
- Any tradeoffs or technical debt introduced?

**Rubber duck the diff** — if I paste a git diff, explain it as:
"Here's what changed and why, written for the architect reviewing it."

### Stepwise mode (when I want full understanding):

If I say "stepwise" or "explain as you go":
1. List every file you plan to touch and what you'll do to each
2. Wait for my approval before making any changes
3. After each file, explain what changed and why before moving on

### Test-first mode (when I say "TDD" or "test first"):

1. Write a failing test that defines done
2. Show me the test — wait for approval
3. Then implement until it passes
4. Never write implementation before I've seen and approved the test

### Reviewing changes from phone / away from desk:

When writing STATUS.md for overnight tasks, include:
- A plain-English summary of what changed (not just file names)
- Any decisions made that I should know about
- What to look at first when I return

---

## Things Claude Should Never Do Without Asking

- Delete or overwrite files
- Commit or push to git
- Make breaking API changes
- Run database migrations
- Install global packages
- Make architectural decisions that affect multiple systems

---

## Compact Instructions

When summarizing this conversation for `/compact`:
- Preserve task state, what's done, what's next, what's blocked
- Keep any architectural decisions and the reasoning behind them
- Note which machine/session this was running on
- Summarize dead ends briefly — don't lose them entirely

---

## Device Setup Checklist

> Paste this section into a new Claude conversation on any machine and say:
> "Help me get set up based on my CLAUDE.md device checklist."
> Claude will walk you through installing and configuring each tool.

### Mac mini (Primary Hub) — Required

- [ ] **Node.js 22+** — `brew install node` (required for both CLIs)
- [ ] **Claude Code** — `npm install -g @anthropic-ai/claude-code`
- [ ] **Copilot CLI** — `npm install -g @github/copilot` then `copilot` → `/login`
- [ ] **cmux** — download from cmux.com (Mac-only, replaces tmux + Ghostty)
- [ ] **tmux** (fallback for SSH sessions) — `brew install tmux`
- [ ] **Ollama** — `brew install ollama` + pull a capable model:
      `ollama pull qwen2.5-coder:32b`
- [ ] **gh CLI** — `brew install gh` then `gh auth login` (for repo creation)
- [ ] **SSH server enabled** — System Settings → General → Sharing → Remote Login
- [ ] **Static local IP or hostname** — set in router or use `.local` hostname
- [ ] **SSH key auth from all devices** — no password prompts
- [ ] **Global CLAUDE.md** — place this file at `~/.claude/CLAUDE.md`
- [ ] **Git configured** — name, email, default branch
- [ ] **VS Code** + GitHub Copilot extension + Claude model selected
- [ ] **VSCodeVim extension** — for evil mode keybindings in VS Code
- [ ] **Starship prompt** — `brew install starship` + add to `~/.zshrc`
- [ ] **Prevent sleep** — System Settings → Energy → disable sleep when on power
- [ ] **Dotfiles repo** — `gh repo create dotfiles --private` and push config

### Windows Laptop — Required

- [ ] **SSH client** — built into Windows 11, or install Git Bash
- [ ] **SSH config** (`~/.ssh/config`) with Mac mini shortcut:
      ```
      Host macmini
        HostName [your-mac-mini-local-ip]
        User [your-username]
        IdentityFile ~/.ssh/id_ed25519
      ```
- [ ] **SSH key** — `ssh-keygen -t ed25519` then copy public key to Mac mini
- [ ] **Windows Terminal** — from Microsoft Store (better tmux experience)
- [ ] **Quick connect alias** — add to PowerShell profile:
      `function mm { ssh macmini -t "tmux attach || tmux new -s main" }`
- [ ] **VS Code** with Remote SSH extension — connect to Mac mini directly
- [ ] **Global CLAUDE.md** — `~/.claude/CLAUDE.md` (copy from Mac mini)

> From Windows: type `mm` to SSH into Mac mini and attach to existing tmux.
> You're now in the same session as if you were sitting at the Mac.

### iPhone / iPad — Required

- [ ] **Blink Shell** or **Prompt 3** — SSH client with tmux support
- [ ] **SSH key** — generate in app or copy from Mac mini
- [ ] **Saved connection** — Mac mini hostname/IP, your username
- [ ] **claude.ai** — mobile app, logged into personal or work account
- [ ] **Saved shortcut** — one tap to SSH → attach tmux session

> From phone: open Blink → connect to Mac mini → `tmux attach -t overnight`
> to check on any running agent task.
> Do NOT use PiKVM terminal for this — SSH is faster and has proper tmux support.

### Apple Vision — Optional but Useful

- [ ] **claude.ai** in browser or app — for reading long diffs, planning sessions
- [ ] **SSH app** if available — or mirror Mac mini display via PiKVM browser
- [ ] **Good for:** reviewing STATUS.md outputs, architectural planning,
      reading long agent logs without being at a desk
- [ ] **PiKVM browser** — useful on Vision for full GUI access to any machine
      when you need to see the desktop (not for terminal work)

### PiKVM — Infrastructure Only (no setup required for AI workflow)

- [ ] **Confirm machines connected:** iMac, Mac mini, Windows laptop
- [ ] **PiKVM web UI accessible** from browser on local network
- [ ] **Bookmark PiKVM URL** on all devices for emergency access
- [ ] **Use case reminder:** recovery and GUI access only — not daily terminal work
- [ ] **SSH keys still preferred** for all routine access — PiKVM is the fallback

---

## Quick Reference — tmux Commands

```bash
# Start a named session
tmux new -s project-name

# Detach (leave running)
Ctrl+b d

# List sessions
tmux ls

# Attach to session
tmux attach -t project-name

# New window in session
Ctrl+b c

# Switch windows
Ctrl+b [number]

# Split pane horizontal
Ctrl+b %

# Split pane vertical
Ctrl+b "
```

---

## Quick Reference — Workflow Modes

**Interactive at desk (VS Code):**
```bash
# Copilot inline in editor — just type, it suggests
# Copilot chat panel — Cmd+Shift+I
# Claude Code in VS Code terminal:
claude
```

**Step away — hand off to CLI:**
```bash
# 1. Get STATUS.md from VS Code Copilot chat first
# 2. Open cmux tab or tmux session
tmux new -s taskname
cd /path/to/project

# 3a. Continue with Copilot CLI (enterprise quota)
copilot
# then: "Read STATUS.md and continue"

# 3b. Or continue with Claude Code
claude
# then: "Read STATUS.md and continue"

# 4. Detach: Ctrl+b d (tmux) or just switch apps (cmux)
```

**Check from phone:**
```bash
ssh macmini -t "tmux attach -t taskname"
# cmux: tap the notification ring tab
```

**Return to VS Code from CLI:**
```bash
/ide          # Copilot CLI — opens current work in VS Code
# Claude Code: just open VS Code in same directory, changes are on disk
```

**Copilot CLI model switching:**
```bash
copilot --model claude-opus-4-6    # launch with specific model
/model                              # switch mid-session
/plan                               # plan before executing
/delegate                           # hand off to background cloud agent
```

**Local LLM fallback (both quotas exhausted):**
```bash
ollama run qwen2.5-coder:32b
# or: OpenClaw/Hermes Agent pointed at Ollama
```

**Overnight parallel (large task):**
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
claude "break this into parallel workstreams and execute"
```
