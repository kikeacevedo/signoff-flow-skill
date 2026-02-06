# Signoff Flow Skill

A Claude Code skill for managing product initiative signoff workflows.

## What it does

Guides users through a structured approval process for product initiatives:

1. **PRD** → BA, Design, Dev approval
2. **UX** → BA, Design approval
3. **Architecture** → Dev approval
4. **Epics & Stories** → BA, Dev approval
5. **Implementation Readiness** → BA, Design, Dev approval

## Installation

### Using npx (Recommended)

```bash
npx skills add https://github.com/kikeacevedo/signoff-flow-skill --skill signoff-flow
```

### Manual Installation

```bash
# Clone the repo
git clone https://github.com/kikeacevedo/signoff-flow-skill.git

# Copy to Claude skills directory
cp -r signoff-flow-skill ~/.claude/skills/signoff-flow
```

## Usage

After installation, just talk to Claude naturally:

- "I want to start a signoff flow"
- "Check my signoff status"
- "Create a new initiative"
- "Help me with approval workflow"

## Prerequisites

The skill will guide you to install these if missing:

- **Git** - Version control
- **GitHub CLI** (`gh`) - For PR creation
- **Atlassian CLI** (`acli`) - For Jira tickets (optional)

## Features

- ✅ Automatic prerequisite checking
- ✅ Project discovery from GitHub
- ✅ Governance configuration
- ✅ Initiative tracking
- ✅ PR creation with reviewers
- ✅ Jira ticket creation
- ✅ Stage progression tracking

## Related

- [Signoff Flow MCP](https://github.com/kikeacevedo/signoff-flow-mcp-v2) - MCP server for Claude Desktop

## License

MIT
