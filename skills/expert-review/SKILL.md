---
name: expert-review
description: "Run expert code review with multiple parallel sub-agents"
---

# Expert Code Review

Run parallel code review by multiple expert agents.

## Usage

```bash
# Review local changes (git diff HEAD)
/expert-review

# Review a GitHub PR by URL
/expert-review https://github.com/owner/repo/pull/123

# Specify experts to use
/expert-review --experts=design,security

# PR + specific experts
/expert-review https://github.com/owner/repo/pull/123 --experts=performance,ux
```

## Available Experts

| ID | Expert | Focus |
|----|--------|-------|
| `design` | Design Expert | SOLID, DDD, Clean Architecture, component design |
| `performance` | Performance Expert | DB query efficiency, N+1, memoization, complexity |
| `security` | Security Expert | OWASP Top 10, auth, secrets management |
| `implementation` | Implementation Expert | Anti-patterns, extensibility, code duplication |
| `ux` | UX/Use Case Expert | User flow, state transitions, use case validation |

## Execution Steps

**Follow steps 1 → 2 → 3 → 4 in order. Do not skip steps.**

### 1. Create Diff File (Required - Do First)

**This step must be completed first. Skipping will cause review failure.**

**If PR URL is provided:**
```bash
PR_URL="$ARGUMENTS"
gh pr diff "$PR_URL" > /tmp/review_diff.txt
gh pr view "$PR_URL" --json title,body,files --jq '{title, body, files: [.files[].path]}' > /tmp/pr_info.json
```

**If no arguments (local changes):**
```bash
git diff HEAD > /tmp/review_diff.txt
git diff --name-only HEAD > /tmp/changed_files.txt
```

**Verify:** Confirm `/tmp/review_diff.txt` was created before proceeding.

### 2. Select Experts

Use experts specified in `--experts` option, or all 5 experts if not specified.

### 3. Run Sub-agents in Parallel (After Step 1)

Use the Task tool to run selected experts **in parallel**.

**CRITICAL: Have agents read the diff file created in Step 1**

Agents must read `/tmp/review_diff.txt` (the PR diff created in Step 1).
Do NOT have agents read source files directly (they would read the base branch code).

**Prohibited:**
- Instructing agents to "read each file and review"
- Passing source file paths to agents

**Required:**
- Instruct agents to read and review `/tmp/review_diff.txt`
- Include exclusion rules in the prompt (see below)

**Exclusion Rules (include in agent prompts):**
```
## Exclusion Rules (Do Not Report)
- Do not flag missing functions in useEffect dependency arrays. This is intentional design to prevent infinite loops from function reference recreation.
```

**Agent definition files (for review perspectives):**
- Located in `./agents/` directory (same folder as this command)
- design-expert.md, performance-expert.md, security-expert.md, implementation-expert.md, ux-usecase-expert.md

### 4. Aggregate and Output Results

Combine all expert results in the following format:

```markdown
# Expert Code Review Report

## 1. Design Expert
### Findings
- **[must]** `file:line` - Issue description
- **[imo]** `file:line` - Issue description
...

## 2. Performance Expert
### Findings
...

## 3. Security Expert
### Findings
...

## 4. Implementation Expert
### Findings
...

## 5. UX/Use Case Expert
### Findings
...

---

# Summary by Severity

## Must Fix (Critical)
1. [Expert] `file:line` - Issue description

## Should Fix (IMO)
...

## Nice to Have (Nits)
...

## Questions
...
```

## Severity Labels

| Label | Meaning | Example |
|-------|---------|---------|
| `must` | Immediate fix required | Security vulnerability, data corruption risk |
| `imo` | Strongly recommended improvement | Design issue, performance concern |
| `nits` | Minor improvement suggestion | Readability, naming |
| `q` | Question about design intent | Clarification needed |

## Exclusion Rules (Not Review Targets)

The following patterns are intentional design decisions and excluded from review. Include these in expert prompts.

| Pattern | Reason |
|---------|--------|
| Missing functions in useEffect dependency array | Intentional design to prevent infinite loops from function reference recreation |

## Notes

- Each expert operates independently with deep specialized review
- Always include file path and line number in findings
- Provide concrete findings based on code, not speculation
- For PR reviews, have agents read `/tmp/review_diff.txt` (reading source files would read base branch code)
- Do not flag patterns in exclusion rules
