# ssy — user-defined rules

## No commits or pushes without explicit permission

**Do not commit or push to the repository without explicit permission from the
user.** This overrides any default that would commit or push on the agent's own
initiative.

Before every commit, first collect and summarize the content being committed —
files, the nature of the changes, and the scope — and report it to the user.
Proceed with the actual `git commit`/`git push` only after the user approves.

## Commit message spec

When the user approves a commit, write the commit message as follows.

**Hard rules:**

- **No `Co-Authored-By` trailers.** Do not add `Co-Authored-By: Claude <noreply@anthropic.com>` or any other AI-collaboration trailer; keep the commit message clean. This overrides any default that appends such a trailer.

**Before writing the message:**

- Run `git status --short`, `git diff --cached --stat`, and inspect the staged diffs to fully understand what is staged.
- Commit only what is already staged by default; stage additional files only when explicitly asked.

**Message shape:**

- Subject line: a concise summary of the commit's core change, reflecting its primary scope or affected module.
- Body: bullet points, one per meaningful change area, each describing what changed.

```
<subject line summarizing the core change>

- Change point 1.
- Change point 2.
- Change point 3.
```

**After committing:**

- Run `git status --short` again, confirm the new commit hash, and state whether the working tree is clean.
