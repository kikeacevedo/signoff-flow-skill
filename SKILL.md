---
name: signoff-flow
description: Use when user mentions signoff, sign-off, approval workflow, initiative approval, BMAD workflow, or wants to manage product initiatives through approval stages (PRD, UX, Architecture, Epics, Readiness). Automatically installs required tools and guides users through complete signoff workflow.
---

# Signoff Flow Workflow

Guide initiatives through a structured approval process using GitHub PRs as gates and Jira tickets for visibility.

## CRITICAL: Auto-Install Behavior

**When a required tool is missing, EXECUTE the installation command directly. Do NOT just show the command to the user.**

This skill is designed for non-technical users who should not need to copy/paste commands.

## Pre-flight Check (ALWAYS RUN FIRST)

When user wants to use signoff flow, execute these checks and auto-install as needed:

### Step 1: Check and Install Homebrew (macOS)

```bash
which brew
```

**If brew is NOT found and OS is macOS:**

Tell the user: "I need to install Homebrew first. This requires your password and may take a few minutes."

Then provide the command for them to run (Homebrew requires interactive installation):
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Wait for confirmation before continuing.

### Step 2: Check and Install Git

```bash
which git
```

**If git is NOT found:**
- macOS: EXECUTE `xcode-select --install` (requires user interaction)
- Tell user to wait for installation to complete

### Step 3: Check and Install GitHub CLI

```bash
which gh
```

**If gh is NOT found, AUTOMATICALLY EXECUTE:**

macOS:
```bash
brew install gh
```

Windows:
```bash
winget install --id GitHub.cli -e
```

**DO NOT just show the command. EXECUTE IT.**

After installation, verify:
```bash
gh --version
```

### Step 4: Check GitHub Authentication

```bash
gh auth status
```

**If NOT authenticated:**

Tell the user: "I need to connect to your GitHub account. This will open a browser window."

Then EXECUTE:
```bash
gh auth login --web
```

This is interactive and requires user action in the browser.

### Step 5: Check and Install Atlassian CLI (for Jira)

```bash
which acli
```

**If acli is NOT found, AUTOMATICALLY EXECUTE:**

```bash
brew tap atlassian/homebrew-acli && brew install acli
```

**DO NOT just show the command. EXECUTE IT.**

After installation, verify:
```bash
acli --version
```

### Step 6: Check Jira Authentication

```bash
acli jira auth status
```

**If NOT authenticated:**

Tell the user: "I need to connect to your Jira account. This will open a browser window."

Then EXECUTE:
```bash
acli jira auth login --web
```

This is interactive and requires user action in the browser.

### Step 7: Check Project

Look for `_bmad-output/governance/governance.yaml` in the current directory.

If not found, help user find or clone a project.

## Summary of Auto-Install Behavior

| Tool | Check Command | Install Command (EXECUTE, don't suggest) |
|------|---------------|------------------------------------------|
| Homebrew | `which brew` | Interactive - guide user |
| Git | `which git` | `xcode-select --install` (interactive) |
| gh CLI | `which gh` | `brew install gh` |
| Atlassian CLI | `which acli` | `brew tap atlassian/homebrew-acli && brew install acli` |

| Auth | Check Command | Auth Command (EXECUTE) |
|------|---------------|------------------------|
| GitHub | `gh auth status` | `gh auth login --web` |
| Jira | `acli jira auth status` | `acli jira auth login --web` |

## Important Instructions for Claude

1. **EXECUTE installation commands directly** - Users are non-technical and should not copy/paste
2. **Wait for each installation to complete** before proceeding to the next check
3. **Only show commands to user** when they require interactive input (passwords, browser auth)
4. **Verify installation** after each install by running the tool with `--version`
5. **Be patient** - installations can take 1-2 minutes

## Workflow Steps (After Prerequisites Met)

### Finding a Project

If no governance found in current directory:

```bash
# List user's GitHub orgs
gh api user/orgs -q '.[].login'

# List recent repos
gh repo list --limit 10
```

Ask user which project they want to work on.

### Cloning a Project

When user specifies a project (e.g., "HALO/my-project"):

```bash
mkdir -p ~/signoff-projects
gh repo clone OWNER/REPO ~/signoff-projects/REPO
cd ~/signoff-projects/REPO
```

### Setting Up Governance

If `_bmad-output/governance/governance.yaml` doesn't exist, ask for:
- BA lead GitHub username(s)
- Design lead GitHub username(s)
- Dev lead GitHub username(s)
- Jira project key

Then CREATE the file:

```yaml
version: 1
groups:
  ba:
    leads:
      github_users: ["username1"]
  design:
    leads:
      github_users: ["username2"]
  dev:
    leads:
      github_users: ["username3"]

jira:
  project_key: "PROJECT"
  issue_types:
    signoff_request: "Task"

signoff_rules:
  prd:
    required_groups: [ba, design, dev]
  ux:
    required_groups: [ba, design]
  architecture:
    required_groups: [dev]
  epics_stories:
    required_groups: [ba, dev]
  readiness:
    required_groups: [ba, design, dev]
```

### Creating an Initiative

Ask for:
- Initiative key (e.g., FEAT-123)
- Title

Create directory structure:
```
_bmad-output/initiatives/<key>/
├── state.yaml
├── timeline.md
└── artifacts/
```

### Advancing Through Stages

For each stage (prd → ux → architecture → epics_stories → readiness):

1. Create artifact stub
2. Create branch: `git checkout -b bmad/<key>/<artifact>`
3. Commit and push
4. Create PR: `gh pr create --title "[BMAD][<key>] <Artifact>" --body "..."`
5. Request reviewers: `gh pr edit --add-reviewer <leads>`
6. Create Jira tickets:
   ```bash
   acli jira workitem create --project "KEY" --type "Task" --summary "[BMAD][<key>][<artifact>] Signoff - GROUP"
   ```

## Example Conversation (Correct Behavior)

```
User: I want to start a signoff flow

Agent: Let me check your setup and install any missing tools...

       [EXECUTES: which gh]
       
       GitHub CLI is not installed. Installing now...
       
       [EXECUTES: brew install gh]
       
       ✅ GitHub CLI installed successfully.
       
       [EXECUTES: gh auth status]
       
       You're not logged into GitHub. I'll open a browser for you to authenticate.
       
       [EXECUTES: gh auth login --web]
       
       Please complete the authentication in your browser...
       
       ✅ Authenticated to GitHub as @username
       
       [EXECUTES: which acli]
       
       Atlassian CLI not installed. Installing now...
       
       [EXECUTES: brew tap atlassian/homebrew-acli && brew install acli]
       
       ✅ Atlassian CLI installed successfully.
       
       All tools ready! Which project do you want to work on?
```

## WRONG Behavior (Do NOT do this)

```
User: I want to start a signoff flow

Agent: GitHub CLI is not installed. To install it, run:
       
       brew install gh
       
       Let me know when you've done that.
```

**THIS IS WRONG.** The agent should EXECUTE the command, not tell the user to run it.
