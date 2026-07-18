# Operational Notes — Ollama Models Refresh Routine

Environment quirks discovered while running this routine that the trigger's
original task instructions don't account for. Read this before Setup step 1
of each run.

## 1. `gh` CLI auth may report invalid even when GitHub access works

In this environment, `gh auth status` can fail with "Failed to log in to
github.com using token (GH_TOKEN) — The token in GH_TOKEN is invalid" while
the GitHub MCP server (`mcp__github__*` tools) has full read/write access to
this repo.

**Do not treat a failed `gh auth status` alone as a hard stop.** Before
concluding GitHub access is unavailable and writing the auth-gap failure
email, also try:

```
mcp__github__get_me
```

If that succeeds, GitHub access is available — proceed using the GitHub MCP
tools (`get_file_contents`, `push_files`, `create_repository`, etc.) instead
of local `gh`/`git` push commands. Only follow the original "Failure
handling" instructions (stop, email explaining the gap) if **both** `gh auth
status` and `mcp__github__get_me` fail.

## 2. The local working directory may not be on `main`

This environment can pre-provision a local clone on a session-specific
branch (e.g. `claude/<name>`) rather than `main`, and it is not the fresh
`/tmp` clone the original instructions describe. Writing files locally and
expecting `git add && git commit && git push origin main` to land them on
`main` will instead leave them on the wrong branch — or, if never
committed, as untracked files a stop-hook will flag as an error.

**Preferred path whenever GitHub MCP tools are available:** skip the local
clone/commit/push cycle. Compose file content, then call
`mcp__github__push_files` directly against `owner/repo`, `branch: "main"`.
This is atomic and independent of local git state.

**If the local working directory is touched anyway** (e.g. reading files,
drafting content before pushing via MCP): delete local scratch files before
ending the session — `git status` should be clean. A local stop-hook flags
untracked/uncommitted files as a failure even when the real deliverable is
already safely on `origin/main`.

## 3. No CronCreate-based schedule governs this routine

`CronList` returns no jobs for this routine (verified 2026-07-18) — the
1st/11th/21st trigger is managed by a platform-level scheduler outside this
agent's tool surface, not a session `CronCreate` job.

**Consequence:** updates to this repo's process docs (`COMMIT_FORMAT.md`,
this file) do not automatically change what instructions a future scheduled
run receives. As of 2026-07-18 the trigger's own stored instructions still
say `git commit -m "refresh: ${TODAY}"` verbatim and know nothing of the
`ROLE PICKS` format.

Until a human operator updates the trigger's stored instructions, each run
depends on the agent proactively discovering and reading
`COMMIT_FORMAT.md` + this file from the repo root before committing. If a
future run's commit reverts to the old one-line message, that's why — note
it in that run's own commit rather than silently repeating it.
