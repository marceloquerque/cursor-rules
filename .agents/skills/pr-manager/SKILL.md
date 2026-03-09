---
name: pr-manager
description: 'Manage GitHub pull requests with `gh` and `git`: create a pull request from the current branch, wait for CI checks, and fetch and organize review comments. Use when the user says things like "create pr", "create the pr", "open a pr", "open the pr", "pull comments", "check for new comments", "check PR comments", or "fetch review comments".'
---

# PR Manager

Use this skill to create a GitHub pull request or to fetch and summarize review comments for an existing pull request.

## Prerequisites

- Use `gh` for all GitHub operations.
- Use `git` for local commit and push operations.
- Work in the current repository unless the user points to a different one.
- Surface `gh` or API failures clearly and suggest `gh auth status` when authentication may be the cause.

## Create PR Workflow

Run this workflow when the user asks to create or open a PR.

1. Verify prerequisites.
   - Run `GIT_EDITOR=true git status` to inspect the working tree.
   - Run `GIT_EDITOR=true git branch --show-current` to get the current branch.
   - Determine the repo default branch with `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`.
   - If that call fails, fall back to `main`, then `master`.
   - If the current branch is the default branch, stop and explain that a PR cannot be created from the default branch.
2. Handle local changes.
   - If unstaged or staged tracked changes exist, commit only the relevant files before creating the PR.
   - Keep commit scope focused. Do not commit unrelated files.
3. Check branch tracking.
   - Run `GIT_EDITOR=true git rev-parse --abbrev-ref @{upstream}`.
   - If no upstream exists, prepare to push with `GIT_EDITOR=true git push -u origin HEAD`.
4. Analyze changes from base to `HEAD`.
   - Run `GIT_EDITOR=true git log <base_branch>..HEAD --oneline`.
   - Run `GIT_EDITOR=true git diff <base_branch>...HEAD --stat`.
   - Run `GIT_EDITOR=true git diff <base_branch>...HEAD` when more detail is needed to draft the PR body.
5. Find a PR template.
   - Check in this order:
     - `.github/pull_request_template.md`
     - `.github/PULL_REQUEST_TEMPLATE.md`
     - `docs/pull_request_template.md`
     - `pull_request_template.md`
   - Also check for multiple templates under `.github/PULL_REQUEST_TEMPLATE/`.
6. Draft PR content.
   - Generate a concise title from the actual changes.
   - Fill every section of the discovered template with concrete details.
   - If no template exists, use this fallback format:
     - `## Summary` with 1-3 bullets
     - `## Test plan` as a checklist
   - Keep the TL;DR to no more than 2 sentences.
7. Push and create the PR.
   - Push with `GIT_EDITOR=true git push -u origin HEAD` when needed.
   - Create the PR with `gh pr create` and a heredoc body.
   - Always return the PR URL.

## Post-PR Checks And Comment Pull

After creating a PR, continue automatically without asking the user.

- Run the wait, polling, and follow-up comment pull in the primary thread.
- Do not move `sleep`, check polling, or the follow-up comment workflow into a background terminal, detached process, or subagent.

1. Wait 5 minutes for CI to start.
   - Use `sleep 300` directly in the primary thread.
2. Poll checks every 30 seconds.
   - Run `gh pr checks <PR_NUMBER> --json name,state,startedAt,completedAt,link`.
   - Treat a check as done when `state` is not `PENDING`, `IN_PROGRESS`, or `QUEUED`.
   - Keep polling every 30 seconds until all checks are done.
   - Show a brief status update each cycle such as `3/5 checks done, waiting...`.
   - Stop after 40 cycles (20 minutes) and report any checks still pending.
3. Report check results.
   - List each check name and final state.
   - Call out any non-success state clearly.
4. Run the full pull-comments workflow against the created PR.
   - If there are no review comments yet, say `No review comments yet` and stop.

## Pull Comments Workflow

Run this workflow when the user asks to pull comments, check for new comments, or review an existing PR discussion.

1. Determine the PR.
   - Use an explicit PR URL or PR number when the user provides one.
   - Otherwise infer PR context from current branch tracking or recent `gh pr` activity.
   - If the PR still cannot be determined, ask: `Which PR? (URL, number, or "current" for this branch)`
2. Fetch all comment sources with `gh`.

```bash
gh pr view <PR> --json reviews,comments,url,title,author --jq '.'
gh api repos/{owner}/{repo}/pulls/{number}/comments --paginate
gh api repos/{owner}/{repo}/issues/{number}/comments --paginate
```

3. Process and organize the fetched data.
   - Group comments by reviewer login.
   - Deduplicate duplicate or near-duplicate comments.
   - Collapse comment threads into a single item with enough context to understand the issue.
4. Categorize severity from the reviewer tone and content.
   - `Critical`: required changes, bugs, security problems, or anything that prevents merge.
   - `Should Fix`: recommended changes, style issues, cleaner approaches, or repeated requests.
   - `Suggestions`: optional ideas, nits, questions, or minor improvements.
5. Explain each comment for non-technical readers.
   - Start with a single plain-language summary paragraph that combines the reviewer request and the practical meaning.
   - `Why it matters`: explain the impact in simple everyday language for a non-technical person and avoid jargon.
   - `How to fix`: explain the likely fix in simple everyday language when the fix is obvious.
   - If a technical term is unavoidable, translate it immediately into plain language.

## Output Format

Number comments sequentially as `C1`, `C2`, `C3`, and so on across all categories so the user can refer to them easily.

```markdown
## PR #123: {title}
By @{author} | {n} reviewers | {total} comments

---

### Critical ({count})

🔴 **#C1** · @reviewer1 · `src/file.tsx:42`
{single plain-language summary of the reviewer comment and what it means in practice}

**Why it matters**: {simple impact explanation for a non-technical person}
**How to fix**: {simple likely fix when obvious}

---

### Should Fix ({count})

🟠 **#C2** · @reviewer2 · `src/api.ts:88`
...

### Suggestions ({count})

🟡 **#C3** · @reviewer3 · `src/index.ts:5`
...

---

## Quick Reference
| # | File | Reviewer | Severity |
|---|------|----------|----------|
| C1 | src/file.tsx:42 | @reviewer1 | 🔴 Critical |
| C2 | src/api.ts:88 | @reviewer2 | 🟠 Should Fix |
| C3 | src/index.ts:5 | @reviewer3 | 🟡 Suggestions |

## Summary
- {critical} critical issues to resolve
- {should_fix} recommended changes
- {suggestions} optional improvements
- Top concerns: {1-2 sentence plain-language summary of the main themes}
```

## Edge Cases

- If the PR has no review comments, say `No review comments yet`.
- If comment threads are very long, summarize the thread into one item and keep only the essential context.
- If the GitHub API fails, show the error and suggest checking `gh auth status`.
