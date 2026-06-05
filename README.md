# Claude Review Commands

Expert code review commands for Claude Code with parallel sub-agents.

## Features

- **`/expert-review`** - Run parallel code review by 5 expert agents (Design, Performance, Security, Implementation, UX)
- **`/fix-review-point`** - Fetch and fix unresolved PR review comments
- **`/fix-sonar`** - Select and fix SonarQube issues

## Installation

### 1. Add Marketplace

```bash
/plugin marketplace add teradonburi/claude-review-commands
```

### 2. Install Plugin

```bash
/plugin install claude-review-commands@teradonburi-marketplace
```

### 3. Verify Installation

```bash
/plugin list
```

You should see `claude-review-commands@teradonburi-marketplace` in the list.

## Usage

### Expert Review

```bash
# Review local changes (git diff HEAD)
/expert-review

# Review a GitHub PR
/expert-review https://github.com/owner/repo/pull/123

# Use specific experts only
/expert-review --experts=design,security

# Combine PR URL and specific experts
/expert-review https://github.com/owner/repo/pull/123 --experts=performance,ux
```

**Available Experts:**

| ID | Expert | Focus |
|----|--------|-------|
| `design` | Design Expert | SOLID, DDD, Clean Architecture |
| `performance` | Performance Expert | N+1, queries, memoization |
| `security` | Security Expert | OWASP Top 10, secrets |
| `implementation` | Implementation Expert | Anti-patterns, DRY |
| `ux` | UX Expert | User flows, state transitions |

### Fix Review Points

```bash
# Fix unresolved PR review comments
/fix-review-point
```

Requires:
- `gh` CLI installed and authenticated
- Current branch has an associated PR

### Fix SonarQube Issues

```bash
# Select and fix SonarQube issues
/fix-sonar
```

Requires:
- SonarQube MCP server configured
- `SONAR_PROJECT_KEY` environment variable set

## Configuration

### SonarQube Project Key

Set the environment variable in your shell profile:

```bash
export SONAR_PROJECT_KEY="your_project_key_here"
```

Or per-project in `.claude/settings.local.json`:

```json
{
  "env": {
    "SONAR_PROJECT_KEY": "your_project_key_here"
  }
}
```

## Updating

To update to the latest version:

```bash
/plugin update claude-review-commands@teradonburi-marketplace
```

## Uninstalling

```bash
/plugin uninstall claude-review-commands@teradonburi-marketplace
```

## Requirements

- Claude Code CLI
- `gh` CLI (for `/fix-review-point` and PR reviews)
- SonarQube MCP server (for `/fix-sonar`)

## License

MIT

## Author

[teradonburi](https://github.com/teradonburi)
