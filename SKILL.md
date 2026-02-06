---
name: signoff-flow
description: Use when user mentions signoff, sign-off, approval workflow, initiative approval, BMAD workflow, or wants to manage product initiatives through approval stages (PRD, UX, Architecture, Epics, Readiness). Guides users through complete signoff workflow including prerequisites setup, project management, and Jira ticket creation.
---

# Signoff Flow Workflow

Guide initiatives through a structured approval process using GitHub PRs as gates and Jira tickets for visibility.

## Overview

This workflow manages the progression of initiatives through planning artifacts:
1. **PRD** (requires: BA, Design, Dev leads)
2. **UX** (requires: BA, Design leads)
3. **Architecture** (requires: Dev lead)
4. **Epics & Stories** (requires: BA, Dev leads)
5. **Implementation Readiness** (requires: BA, Design, Dev leads)

**Key principles:**
- Repo is source of truth (state files tracked in git)
- PR-centric signoff (approval = PR approval + merge)
- Lead-only signoff (only configured leads count)
- Jira is visibility only (tickets mirror signoffs, not canonical)

## Pre-flight Check

**Before doing anything else, check prerequisites:**

### 1. Check Tools

```bash
# Check Git
which git

# Check GitHub CLI
which gh

# Check Atlassian CLI
which acli
```

**If gh is NOT installed:**
```
GitHub CLI Required. Install with:
macOS: brew install gh
Windows: winget install --id GitHub.cli
```

**If acli is NOT installed:**
```
Atlassian CLI Required for Jira. Install with:
brew tap atlassian/homebrew-acli
brew install acli
```

### 2. Check Authentication

```bash
# GitHub
gh auth status

# Jira
acli jira auth status
```

If not authenticated, guide the user:
- GitHub: `gh auth login`
- Jira: `acli jira auth login --web`

### 3. Check Project

Look for `_bmad-output/governance/governance.yaml` in the current directory.

If no governance found, help user find a project:

```
## No signoff workflow found

Let me help you find a project:

### Your GitHub Organizations
[Run: gh api user/orgs -q '.[].login']

### Your Recent Repositories  
[Run: gh repo list --limit 5]

### Options:
1. Clone a project (e.g., "HALO/my-project")
2. Set up signoff in current directory
```

## Workflow Steps

### Step 1: Setup Governance

If governance missing, ask for:
- BA lead GitHub username(s)
- Design lead GitHub username(s)  
- Dev lead GitHub username(s)
- Jira project key

Create `_bmad-output/governance/governance.yaml`:

```yaml
version: 1
groups:
  ba:
    leads:
      github_users: ["<lead1>"]
  design:
    leads:
      github_users: ["<lead1>"]
  dev:
    leads:
      github_users: ["<lead1>"]

jira:
  project_key: "<PROJECT_KEY>"
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

### Step 2: Create Initiative

Ask for initiative key (e.g., FEAT-123) and title.

Create:
- `_bmad-output/initiatives/<key>/state.yaml`
- `_bmad-output/initiatives/<key>/timeline.md`
- `_bmad-output/initiatives/<key>/artifacts/`

### Step 3: Advance Through Stages

For each stage (prd → ux → architecture → epics_stories → readiness):

1. Create stub artifact at `artifacts/<ARTIFACT>.md`
2. Create branch: `bmad/<key>/<artifact>`
3. Commit and push
4. Create GitHub PR with reviewers
5. Create Jira tickets using acli:
   ```bash
   acli jira workitem create \
     --project "<PROJECT_KEY>" \
     --type "Task" \
     --summary "[BMAD][<key>][<artifact>] Signoff required — <GROUP>"
   ```
6. Update state.yaml

### Step 4: Check Status

When asked for status:
- Read initiative's state.yaml
- Report current step
- If PR merged, ready to advance
- If PR open, show pending approvals

## File Locations

```
_bmad-output/
├── governance/
│   └── governance.yaml
└── initiatives/
    └── <key>/
        ├── state.yaml
        ├── timeline.md
        └── artifacts/
            ├── PRD.md
            ├── UX.md
            ├── ARCHITECTURE.md
            ├── EPICS_AND_STORIES.md
            └── IMPLEMENTATION_READINESS.md
```

## Example Conversation

```
User: I want to start a signoff flow
Agent: Let me check your setup...
       ✅ Git installed
       ✅ GitHub CLI installed  
       ✅ Authenticated to GitHub
       ❌ Atlassian CLI not installed
       
       To install: brew tap atlassian/homebrew-acli && brew install acli

User: Done, installed it
Agent: ✅ Atlassian CLI installed
       ❌ Not authenticated to Jira
       
       Run: acli jira auth login --web

User: Authenticated
Agent: ✅ All set! Which project do you want to work on?

User: Clone HALO/feature-payments
Agent: ✅ Cloned to ~/signoff-projects/feature-payments
       ✅ Found governance configuration
       
       Current status: No active initiatives
       Say "create new initiative" to start one.
```
