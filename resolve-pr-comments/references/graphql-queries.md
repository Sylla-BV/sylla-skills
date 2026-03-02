# GraphQL Queries for PR Review Threads

## Fetch Review Threads

Retrieve all review threads for a pull request. Replace `OWNER`, `REPO`, and `PR_NUMBER` with actual values.

```bash
gh api graphql -F owner='OWNER' -F repo='REPO' -F pr=PR_NUMBER -f query='
query($owner: String!, $repo: String!, $pr: Int!) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $pr) {
      reviewThreads(first: 100) {
        nodes {
          id
          isResolved
          path
          line
          startLine
          diffSide
          comments(first: 50) {
            nodes {
              id
              body
              author { login }
              createdAt
            }
          }
        }
      }
    }
  }
}'
```

### Response Shape

```json
{
  "data": {
    "repository": {
      "pullRequest": {
        "reviewThreads": {
          "nodes": [
            {
              "id": "PRRT_...",
              "isResolved": false,
              "path": "src/file.ts",
              "line": 42,
              "startLine": null,
              "diffSide": "RIGHT",
              "comments": {
                "nodes": [
                  {
                    "id": "PRRC_...",
                    "body": "Consider using a constant here",
                    "author": { "login": "reviewer" },
                    "createdAt": "2025-01-15T10:00:00Z"
                  }
                ]
              }
            }
          ]
        }
      }
    }
  }
}
```

Filter for threads where `isResolved` is `false`.

## Reply to Thread

Post a reply comment on a review thread before resolving it. Replace `THREAD_NODE_ID` with the thread's `id` and `REPLY_BODY` with the reply text.

```bash
gh api graphql -f query='
mutation($threadId: ID!, $body: String!) {
  addPullRequestReviewThreadReply(input: {
    pullRequestReviewThreadId: $threadId,
    body: $body
  }) {
    comment {
      id
      body
    }
  }
}' -F threadId='THREAD_NODE_ID' -F body='REPLY_BODY'
```

### Reply Guidelines

- Keep replies to one or two sentences
- Reference what was changed: "Fixed in <commit> - renamed to `betterName`"
- For refactoring: "Addressed by refactoring X into Y, which also resolves the concern about Z"
- For removals: "This code was removed as part of the refactor in <commit>"

## Resolve Thread

Mark a review thread as resolved. Replace `THREAD_ID` with the thread's `id`.

```bash
gh api graphql -f query='
mutation($threadId: ID!) {
  resolveReviewThread(input: { threadId: $threadId }) {
    thread {
      id
      isResolved
    }
  }
}' -F threadId='THREAD_ID'
```

## Unresolve Thread

If a thread was resolved by mistake, unresolve it. Replace `THREAD_ID` with the thread's `id`.

```bash
gh api graphql -f query='
mutation($threadId: ID!) {
  unresolveReviewThread(input: { threadId: $threadId }) {
    thread {
      id
      isResolved
    }
  }
}' -F threadId='THREAD_ID'
```
