# Operational Notes — Ollama Models Refresh Routine

Environment quirks and execution rules discovered while actually running
this routine that the trigger's original task instructions don't account
for. Read this before Setup step 1 of each run.

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
status` and `mcp__github__get_me` fail — see §2 below for what to do if the
GitHub MCP server itself is mid-reconnect rather than truly down.

## 2. MCP servers can disconnect and reconnect mid-run

Observed directly in this session (2026-07-18): the GitHub MCP server
disconnected mid-conversation — a system-reminder announced `mcp__github__*`
tools as unavailable — and reconnected several turns later, announced by a
follow-up system-reminder. §1's `gh`-auth fallback assumes the GitHub MCP
server is always reachable; it says nothing about the MCP *connection
itself* dropping independent of the `gh` CLI auth issue.

**Correct response when a needed MCP tool is reported unavailable:**
1. Check what the system-reminder actually says: a server "still
   connecting" is worth waiting on; a server marked fully "disconnected"
   means its tools are gone until a *future* reconnection announcement.
2. If "still connecting": call `ToolSearch` for the tool. Its own
   description states it "will wait for connecting servers and search
   their tools once available" — this is the correct wait mechanism. Do
   not `sleep`-poll or repeatedly retry the tool call directly.
3. If fully "disconnected": do not spin-wait. Reconnection is
   push-notified via a later system-reminder, not something pollable.
   Continue with whatever other work is possible (research, drafting
   content, local file prep) and retry the blocked operation once a
   reconnection reminder appears in context.
4. If the run would otherwise conclude with the GitHub MCP server still
   disconnected **and** `gh` CLI also failing (§1) — that is a genuine
   dual-fallback failure, not a transient blip. Follow the original
   "Failure handling" instructions: capture the error and note it
   prominently in whatever output channel is still reachable (the email
   draft, if Gmail MCP is up, since GitHub itself can't be used to log it).

## 3. The local working directory may not be on `main`

This environment can pre-provision a local clone on a session-specific
branch (e.g. `claude/<name>`) rather than `main`, and it is not the fresh
`/tmp` clone the original instructions describe. Writing files locally and
expecting `git add && git commit && git push origin main` to land them on
`main` will instead leave them on the wrong branch — or, if never
committed, as untracked files a stop-hook will flag as an error.

**Preferred path whenever GitHub MCP tools are available:** skip the local
clone/commit/push cycle entirely. Compose file content in the scratchpad
directory (never the local repo checkout), then call
`mcp__github__push_files` directly against `owner/repo`, `branch: "main"`.
This is atomic, independent of local git state, and leaves the local
working tree untouched — verified clean across multiple runs this session
when this path was followed.

**If the local working directory is touched anyway** (e.g. reading files
for comparison): delete local scratch files before ending the session —
`git status` should be clean. A local stop-hook flags untracked/uncommitted
files as a failure even when the real deliverable is already safely on
`origin/main`.

## 4. No CronCreate-based schedule governs this routine

`CronList` returns no jobs for this routine (verified 2026-07-18) — the
1st/11th/21st trigger is managed by a platform-level scheduler outside this
agent's tool surface, not a session `CronCreate` job.

**Consequence:** updates to this repo's process docs (`COMMIT_FORMAT.md`,
this file, `README.md`) do not automatically change what instructions a
future scheduled run receives. As of 2026-07-18 the trigger's own stored
instructions still say `git commit -m "refresh: ${TODAY}"` verbatim and know
nothing of the `ROLE PICKS` format. `README.md` now carries an unmissable
pointer to these docs as a mitigation, but it only helps if the agent
executing the trigger actually reads the repo root before acting — it is
not a guarantee.

Until a human operator updates the trigger's stored instructions, each run
depends on the agent proactively discovering and reading `README.md` /
`COMMIT_FORMAT.md` / this file before committing. If a future run's commit
reverts to the old one-line message, that's why — note it in that run's own
commit rather than silently repeating it.

## 5. Pre-publish: verify every Ollama pull tag against the live library

Model resident sizes and Ollama tag *names* sourced from secondary blog
posts and comparison articles are frequently wrong or stale — they get
rewritten and re-aggregated from other summaries and drift from the actual
`ollama.com/library/<model>/tags` page.

**Concrete example from this repo's history:** the 2026-07-18 research
(commit `bea2b59`) listed the code-implementer smaller pick under the tag
`devstral-2:small` — sourced from a blog post describing "Devstral Small
2." The real Ollama library tag is a *differently-named model entirely*,
`devstral-small-2` (not a `:small` tag under a `devstral-2` model).
`ollama pull devstral-2:small` would have failed outright. The same
verification pass also found `llama4:scout`'s resident size understated by
12 GB (55 GB stated vs. 67 GB actual per the official tags page) and
`gemma4:e4b`'s understated by roughly 2x (5 GB stated vs. 9.6 GB actual).

**Rule:** before publishing the "Verification Checklist" section of any
monthly doc, for every `ollama pull <tag>` line, search or fetch directly
for `ollama.com/library/<model-name>/tags` (or the model's own library
page) and confirm both the tag string and the resident size against that
page — not against a blog's restatement of it. If the tag doesn't appear on
the official tags page, the model name or tag format is wrong and must be
corrected before publishing, not left as an assumption.

## 6. Gmail is draft-only — no send capability, and same-day duplicate handling

This environment's Gmail MCP integration exposes `create_draft`,
`list_drafts`, `get_message`/`get_thread`, `search_threads`, and label
operations — **there is no `send_message`/`send_draft` tool.** The original
routine instructions already specify "create a draft (not send)," so this
is not a deviation from the intended design. Worth stating explicitly
anyway: even if a future environment adds send capability, switching from
draft to auto-send is a deliberate policy change (unreviewed content lands
directly in the recipient's inbox) and should be confirmed with the
operator before being adopted — never inferred from tool availability
alone.

**Same-day duplicate drafts:** if the routine fires more than once on the
same calendar day (e.g. a manual test/validation re-run), each run creates
a new draft with the spec's standard subject
(`Ollama models refresh — ${TODAY}`), which is indistinguishable from an
earlier same-day draft at a glance. Before creating a draft, call
`list_drafts` (or `search_threads` with
`in:draft subject:"Ollama models refresh — ${TODAY}"`) to check for an
existing same-day draft. If one exists, make the new draft's subject and
opening line explicitly reference it as a follow-up/re-run rather than
leaving two identically-titled drafts with no indication of their
relationship.
