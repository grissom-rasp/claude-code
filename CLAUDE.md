# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains the official Claude Code plugins and GitHub automation for the Claude Code CLI tool. The repo is a distribution of plugins, configuration examples, and maintenance scripts rather than a traditional source code build project.

## Common Commands

### Running TypeScript Scripts (Bun runtime)
```bash
bun run scripts/<script-name>.ts
```

Examples:
- `bun run scripts/auto-close-duplicates.ts` - Auto-closes duplicate GitHub issues
- `bun run scripts/issue-lifecycle.ts` - Manages issue lifecycle
- `bun run scripts/sweep.ts` - Sweeps stale issues
- `bun run scripts/backfill-duplicate-comments.ts` - Backfills duplicate comments

### Bash Utility Scripts
```bash
./scripts/gh.sh <subcommand> <arguments>
```

Wrapper around gh CLI that only allows specific subcommands and flags, scoped to the current repository.

## Architecture

### Plugin System

Claude Code has a sophisticated plugin architecture. Plugins extend functionality through:

- **Commands**: Slash commands (e.g., `/commit`, `/feature-dev`)
- **Agents**: Specialized AI agents for specific tasks (e.g., `comment-analyzer`, `code-reviewer`)
- **Skills**: Auto-invoked capabilities for specific contexts (e.g., `frontend-design`)
- **Hooks**: Event handlers (`PreToolUse`, `SessionStart`, `Stop`)
- **MCP Integration**: External tool configurations via `.mcp.json`

#### Plugin Structure
```
plugin-name/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata (name, version, author)
├── commands/                # Slash commands as .md files
├── agents/                  # Agent definitions as .md files
├── skills/                  # Agent Skills as .md files
├── hooks/                   # Hook definitions (hooks.json + executable scripts)
├── .mcp.json                # External tool configuration (optional)
└── README.md                # Plugin documentation
```

#### Command Format (.md files)
Commands are markdown files with frontmatter:
```markdown
---
allowed-tools: Bash(git add:*), Bash(git status:*)
description: Create a git commit
---
## Context
- Current git status: !`git status`
## Your task
...
```

#### Agent Format (.md files)
Agents have frontmatter defining name, description, model, color:
```markdown
---
name: comment-analyzer
description: Use this agent when...
model: inherit
color: green
---
You are a specialized agent...
```

#### Skill Format (SKILL.md files)
Skills are defined in `skills/<skill-name>/SKILL.md` with frontmatter:
```markdown
---
name: frontend-design
description: Create distinctive interfaces...
---
This skill guides creation of...
```

#### Hook Format
Hooks are defined in `hooks/hooks.json`:
```json
{
  "description": "Security reminder hook...",
  "hooks": {
    "PreToolUse": [{
      "hooks": [{
        "type": "command",
        "command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/script.py"
      }],
      "matcher": "Edit|Write|MultiEdit"
    }]
  }
}
```

### Official Plugins (13 total)

Located in `plugins/` directory. Key plugins:

- **agent-sdk-dev**: `/new-sdk-app` command for new Agent SDK projects
- **commit-commands**: `/commit`, `/commit-push-pr`, `/clean_gone` for git workflows
- **feature-dev**: `/feature-dev` guided 7-phase development workflow
- **pr-review-toolkit**: `/pr-review-toolkit:review-pr` with specialized review agents
- **code-review**: `/code-review` with 5 parallel Sonnet agents
- **plugin-dev**: `/plugin-dev:create-plugin` for building plugins
- **security-guidance**: PreToolUse hook for security reminders
- **frontend-design**: Auto-invoked skill for frontend work
- **hookify**: `/hookify` for creating custom hooks
- **ralph-wiggum**: `/ralph-loop` for iterative development

### GitHub Automation

Workflows in `.github/workflows/` use Bun to run TypeScript scripts:

- **auto-close-duplicates.yml**: Daily cron to close duplicate issues
- **issue-lifecycle-comment.yml**: Comments when issues are labeled
- **claude.yml**: Main workflow that runs Claude Code on @claude mentions in issues/PRs
- **claude-dedupe-issues.yml**: Dedupes GitHub issues using Claude
- **claude-issue-triage.yml**: Triages issues using Claude

### Development Environment

The repository includes a VS Code Dev Container configuration (`.devcontainer/`):

- Runtime: Node 20 on Linux
- Shell: zsh with powerline10k
- Tools: git, gh, delta, fzf, jq, iptables/ipset for firewall
- Extensions: anthropic.claude-code, ESLint, Prettier, GitLens
- Network restrictions: Firewall script controls outbound network access

### Settings Examples

Located in `examples/settings/` for organization-wide deployments:

- **settings-lax.json**: Blocks `--dangerously-skip-permissions`, blocks marketplaces
- **settings-strict.json**: Adds bash sandbox, blocks user hooks, denies web tools
- **settings-bash-sandbox.json**: Bash tool must run inside sandbox

These are starting points for managed settings. See [settings documentation](https://code.claude.com/docs/en/settings) for full reference.

### Marketplace Configuration

`.claude-plugin/marketplace.json` defines the bundled plugins available in the official marketplace. Each plugin entry includes name, description, version, author, source path, and category.

## Key Files

- `.claude-plugin/marketplace.json`: Plugin marketplace definition
- `.devcontainer/devcontainer.json`: Dev container configuration
- `.devcontainer/Dockerfile`: Container image definition
- `.devcontainer/init-firewall.sh`: Network firewall initialization
- `scripts/`: TypeScript scripts for GitHub automation
- `plugins/`: Official Claude Code plugins
- `examples/settings/`: Settings configuration examples
- `.github/workflows/`: GitHub Actions automation
- `.github/ISSUE_TEMPLATE/`: GitHub issue templates

## TypeScript Script Patterns

Scripts in `scripts/` follow a common pattern:

```typescript
#!/usr/bin/env bun

interface GitHubIssue { ... }
interface GitHubComment { ... }

async function githubRequest<T>(endpoint: string, token: string, method: string = 'GET', body?: any): Promise<T> {
  const response = await fetch(`https://api.github.com${endpoint}`, { method, headers, body });
  // ...
}
```

They use the GitHub API token from `GITHUB_TOKEN` environment variable.
