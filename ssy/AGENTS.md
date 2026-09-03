# ssy — user-defined rules

## Project background

This repository is a fork of the official DeepSeek Harness project at
<https://github.com/deepseek-ai/deepseek-harness.git>. The branching model is
split by purpose:

- **`master`** tracks the official upstream. It stays on the upstream history and
  is used only to sync official updates; do not land personal changes here.
- **`dev`** carries personal 魔改 (custom modifications) on top of upstream. It
  is the working branch where the user's own changes accumulate.
- **The locally run `dsh` is built from the `dev` branch source**, not from a
  globally installed published package. Treat the `dev` checkout as the source
  of truth for the runtime the user actually runs: build it
  (`pnpm install && pnpm run build`) and launch from it (for example
  `pnpm dsh --profile <name> ...`), rather than suggesting `npm install -g
  @deepseek-ai/dsh`.

When a task touches harness behavior, assume the `dev` branch is in effect and
that both harness internals and third-party plugins developed in separate
checkouts (e.g. the superpowers primer at
`~/Documents/deepseek-harness/superpowers-ssy/dsh-primer`) are in scope for 魔改.

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

## Daily operations cheat-sheet

Two checkouts are in play; keep their roles distinct:

- **Harness checkout**: `~/Documents/deepseek-harness-ssy`, branch `dev`. This is
  the runtime the user actually runs. `origin` = the user's fork
  `pandas1911/deepseek-harness-ssy` (push target); `upstream` =
  `deepseek-ai/deepseek-harness` (fetch-only sync source). Never push to
  `upstream` (and there is no write access anyway).
- **Local link plugin**: the superpowers primer at
  `~/Documents/deepseek-harness/superpowers-ssy/dsh-primer`, installed into both
  the `web` and `headless` profiles as a `link:` symlink
  (`@local/dsh-superpowers-primer`). Its devDeps `link:` the local harness
  packages so it shares the `dev` core.

### Run dsh (from the harness `dev` source, never a global install)

```sh
cd ~/Documents/deepseek-harness-ssy
pnpm install && pnpm run build          # first time, or after harness dependency/source changes
pnpm dsh web                            # = pnpm dsh --profile web; boots Web UI at http://127.0.0.1:3080
pnpm dsh --profile headless "task"      # run one task headless and exit
pnpm dsh --version                      # sanity check (prints 0.1.1-rc.1)
```

`pnpm dsh` runs via `node --import tsx/esm apps/cli/src/bin.ts`, so edits to CLI
source take effect without a rebuild. Profile **bundles** load their own `lib/`,
so a plugin edit still needs `pnpm build` in the plugin checkout. `DEEPSEEK_API_KEY`
must be available (env, root `.env`, or `~/.dsh/settings.yaml`) for agent work.

For a short `dsh` shell alias instead of `pnpm dsh`, add to the shell rc:
`alias dsh='pnpm -C ~/Documents/deepseek-harness-ssy dsh'`. Do **not**
`npm install -g @deepseek-ai/dsh` — that installs an independent published copy
and diverges from the `dev` source.

### Manage web-profile plugins

```sh
cd ~/Documents/deepseek-harness-ssy
pnpm dsh plugin --profile web list
pnpm dsh plugin --profile web add @nanmicoder/dsh-agent-teams@latest               # install official plugin from npm
pnpm dsh plugin --profile web add link:/Users/sys/Documents/deepseek-harness/superpowers-ssy/dsh-primer  # symlink local primer
pnpm dsh plugin --profile web remove @nanmicoder/dsh-agent-teams                  # drop a plugin (source untouched)
pnpm dsh --profile web --dump-config                                              # print composed config
```

### Plugin 魔改 loop (local link plugins, e.g. the primer)

```sh
cd ~/Documents/deepseek-harness/superpowers-ssy/dsh-primer   # dev branch
# edit src...
pnpm build                              # rebuild lib/; profiles pick it up live via the link, no reinstall
```

### Sync harness from upstream (keep `master` clean, then merge-forward `dev`)

```sh
cd ~/Documents/deepseek-harness-ssy
git fetch upstream
git checkout master && git merge upstream/master    # master tracks upstream only
git checkout dev && git merge master                # bring upstream into the 魔改 branch
# resolve conflicts, then (with explicit user approval) push to origin fork
```

### After upstream sync — check the superpowers `dsh-tools` reference

After merging upstream into `dev`, the dsh tool surface may have changed
(renamed/added/removed tools, config-default tool names like `subagent` /
`workflow`). The superpowers fork at `~/Documents/deepseek-harness/superpowers-ssy`
ships a dsh platform reference:
`skills/using-superpowers/references/dsh-tools.md`. Its **tool table is the
only part that drifts** — re-check it against the current
`packages/*/tools/*` and update any changed tool name or surface. The rest
(no native worktree tool → git fallback, `~/.dsh/skills/` path, plan mode,
subagent `spawn`/`send_message`/`report` flow, environment detection) is
stable structural guidance and rarely needs touching.

This check is best-effort, not blocking: the doc explicitly tells the model
to trust its real runtime tool list over the table, so a stale entry
self-degrades safely rather than misbehaving.
