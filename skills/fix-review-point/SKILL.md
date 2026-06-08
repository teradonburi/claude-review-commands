---
name: fix-review-point
description: "Fix unresolved PR review comments"
---

# Fix Review Points

Address unresolved review comments on the current PR.

## Workflow

1. Fetch unresolved review comments using the command below
2. Understand the content of unresolved comments
3. Present a summary of each comment to the user with proposed actions
4. Wait for user approval before making any changes
5. Fix the code based on approved actions (do NOT reply with PR comments unless explicitly requested)

## GitHub CLI Command

Use this command to fetch unresolved review comments:

```bash
OWNER_REPO=$(gh repo view --json nameWithOwner --jq '.nameWithOwner')
OWNER=$(echo $OWNER_REPO | cut -d'/' -f1)
REPO=$(echo $OWNER_REPO | cut -d'/' -f2)
PR_NUMBER=$(gh pr view --json number --jq '.number')

gh api graphql -f query="
query {
  repository(owner: \"${OWNER}\", name: \"${REPO}\") {
    pullRequest(number: ${PR_NUMBER}) {
      number
      title
      url
      state
      author {
        login
      }
      reviewRequests(first: 20) {
        nodes {
          requestedReviewer {
            ... on User {
              login
            }
          }
        }
      }
      reviewThreads(last: 20) {
        edges {
          node {
            isResolved
            isOutdated
            path
            line
            comments(last: 20) {
              nodes {
                author {
                  login
                }
                body
                url
                createdAt
              }
            }
          }
        }
      }
    }
  }
}" --jq '
  .data.repository.pullRequest as $pr |
  {
    pr_number: $pr.number,
    title: $pr.title,
    url: $pr.url,
    state: $pr.state,
    author: $pr.author.login,
    requested_reviewers: [.data.repository.pullRequest.reviewRequests.nodes[].requestedReviewer.login],
    unresolved_threads: [
      $pr.reviewThreads.edges[] |
      select(.node.isResolved == false) |
      {
        path: .node.path,
        line: .node.line,
        is_outdated: .node.isOutdated,
        comments: [
          .node.comments.nodes[] |
          {
            author: .author.login,
            body: .body,
            url: .url,
            created_at: .createdAt
          }
        ]
      }
    ]
  }
'
```

## Output

The command returns:
- PR metadata (number, title, URL, state, author)
- List of requested reviewers
- Unresolved review threads with:
  - File path and line number
  - Whether the comment is outdated
  - All comments in the thread with author, body, URL, and timestamp

## Notes

- Requires `gh` CLI to be installed and authenticated
- Must be run from within a git repository with an associated PR
- Only shows unresolved threads (resolved threads are filtered out)

## Important

- **Do NOT automatically post PR comments** without explicit user approval
- **Do NOT make code changes** without presenting the plan to the user first
- For each review comment, present:
  1. The comment content and location
  2. Your proposed fix or response
  3. Ask user to approve before proceeding
- If the comment is a [nit] or question, ask user if they want to fix it or skip
