---
name: git-worktree-workflow
description: Use when starting or resuming work on a code-related Paperclip issue that requires a feature branch and a pull request. Handles worktree creation from the latest main, dependency installation, commit conventions, PR creation, status updates, and graceful failure handling. Safe for parallel agents working on the same repository.
---

# Git Worktree Workflow

Work on a Paperclip issue in an isolated git worktree, branched from the latest `main`, so multiple agents can operate on the same repository without conflicts. Each task gets its own worktree and feature branch; the workflow is idempotent across heartbeats.

## When to use this skill

Activate this skill whenever you start or resume work on a Paperclip issue that involves code changes in a project with a git repository configured as its workspace. Do not use it for non-code tasks (pure documentation coordination, research, planning).

## Environment expectations

These environment variables are injected by Paperclip at run time:

- `PAPERCLIP_TASK_ID` — the issue triggering this run
- `PAPERCLIP_AGENT_ID`, `PAPERCLIP_COMPANY_ID`, `PAPERCLIP_API_URL`, `PAPERCLIP_API_KEY` — for Paperclip API calls
- `PAPERCLIP_WAKE_REASON` — why this heartbeat was triggered

The project's primary workspace is a git repository with `origin` pointing to the canonical remote, and `main` as the integration branch. If the integration branch is called something else (e.g. `develop`), adapt the references in Step 1 accordingly.

## Workflow

### Step 0: Check for an existing worktree

A previous heartbeat may have already created a worktree for this task. Do not recreate it — resume instead.

Sync remote refs first, then inspect existing worktrees:

```bash
git fetch origin --prune
git worktree list
```

If any entry matches `paperclip-$PAPERCLIP_TASK_ID`, `cd` into it and skip to Step 4. Otherwise, proceed with setup.

### Step 1: Create the worktree

Branch from the latest `origin/main`:

```bash
git worktree add ../worktrees/paperclip-$PAPERCLIP_TASK_ID \
  -b feature/paperclip-$PAPERCLIP_TASK_ID origin/main
```

### Step 2: Enter the worktree

```bash
cd ../worktrees/paperclip-$PAPERCLIP_TASK_ID
```

### Step 3: Install dependencies

Worktrees share `.git` but not `node_modules` or .NET build artifacts. Restore both the frontend and backend before starting work:

- **Frontend (Vue.js):** `bun install` in the frontend directory
- **Backend (C#/.NET):** `dotnet restore` in the backend solution/project directory

If either restore step fails, go to **Failure handling**.

### Step 4: Do the work

Implement the changes required by the issue. Test locally before committing. Commit in focused, well-described chunks.

Commit message format:

```
<type>(<scope>): <short description>

<optional longer body>

Refs: PAPERCLIP-$PAPERCLIP_TASK_ID
```

`<type>` should be one of `feat`, `fix`, `refactor`, `docs`, `test`, `chore`. Avoid placeholder messages like "wip" or "fix stuff" in final commits.

### Step 5: Push and open a PR

```bash
git push -u origin feature/paperclip-$PAPERCLIP_TASK_ID
gh pr create \
  --title "<matches main commit subject>" \
  --body "<summary of changes>

Paperclip issue: $PAPERCLIP_API_URL/issues/$PAPERCLIP_TASK_ID"
```

### Step 6: Update Paperclip status

After the PR is open, patch the issue status to `in_review`. Use `done` only *after* the PR is merged — not before.

```bash
curl -X PATCH "$PAPERCLIP_API_URL/api/issues/$PAPERCLIP_TASK_ID" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"status": "in_review"}'
```

Then post a comment on the issue containing the PR link, so reviewers can jump straight from the Paperclip dashboard to the PR.

## Failure handling

If any step fails — dependency install errors, unexpected branch conflicts, push rejected, tests failing in unrelated areas — **do not retry in a loop**. Instead:

1. Patch the issue status to `blocked`
2. Post a comment describing exactly what failed, what you tried, and what would be needed to unblock
3. Exit the heartbeat cleanly

Typical blocker scenarios:

- `pnpm install` fails due to a lockfile conflict → blocked with conflict details
- `git push` rejected because branch diverged → blocked, suggest rebase approval
- Tests fail in areas unrelated to the change → blocked with test output

Do not post the same blocker comment repeatedly across heartbeats; only comment when there is new information.

## Out of scope for this skill

- **Review handoff to humans.** If a board user asks to review the PR manually, reassign the issue to them with `assigneeAgentId: null` and the requesting user's ID, and set status to `in_review`.
- **Follow-up issues on the same branch.** If a new issue is a direct follow-up on previous work and should reuse the existing worktree, have the caller set `parentId` (child issues inherit execution workspace automatically) or pass `inheritExecutionWorkspaceFromIssueId` explicitly. Do not branch off `main` again for follow-ups.

## Notes

- Worktree path is `../worktrees/paperclip-<task-id>` relative to the primary repo checkout. Make sure the parent `worktrees/` directory is gitignored where appropriate.
- Branch name `feature/paperclip-<task-id>` is deliberate: the `feature/` prefix keeps git history readable as normal team development, while the embedded task ID gives full traceability back to Paperclip.
