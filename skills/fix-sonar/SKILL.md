---
description: Fix SonarQube issues detected in the current PR
---

# Fix SonarQube Issues

Select and fix issues detected by SonarQube for the current PR.

## Prerequisites

- SonarQube MCP server must be configured
- Environment variable `SONAR_PROJECT_KEY` must be set

## Workflow

1. Fetch issues for current PR using SonarQube MCP
2. Display the list and let user select which issues to fix
3. Fix only the selected issues

## Fetching Issues

Use SonarQube MCP tool `mcp__sonarqube__search_sonar_issues_in_projects`:

```
projectKey: $SONAR_PROJECT_KEY (from environment variable)
pullRequestId: $(gh pr view --json number -q '.number')
issueStatuses: ["OPEN", "CONFIRMED"]
```

If `SONAR_PROJECT_KEY` is not set, ask the user to provide it.

## Display Format

Show the list in this format:

```
PR #XXXX SonarQube Issues (N items)

1. [HIGH] filename:line
   Message: Do not use Array index in keys
   Rule: typescript:S6479

2. [MEDIUM] filename:line
   Message: ...

3. [LOW] filename:line
   Message: ...
```

## User Selection

Use AskUserQuestion to let user select issues to fix:
- Enable multi-select (multiSelect: true)
- Show each issue as an option
- Only fix user-selected issues

## After Fixing

Report the list of fixed issues when complete.

## Environment Setup

Add to your shell profile or `.envrc`:

```bash
export SONAR_PROJECT_KEY="your_project_key_here"
```

Or set per-project in `.claude/settings.local.json`:

```json
{
  "env": {
    "SONAR_PROJECT_KEY": "your_project_key_here"
  }
}
```
