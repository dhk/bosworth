# Wyatt

**A Marshal for cross-tool research: takes submissions from Claude, ChatGPT,
and Perplexity, validates them against the brief's contract, and marshals
the result into GitHub as a PR.**

## 1. What observation or need does this enable?

Cross-tool research briefs (one brief, answered independently by several
LLMs, then synthesized — the multi-source pattern this repo already uses
via `commission-research`) currently require manually copying each tool's
output between apps and into a git repo by hand. That copy-paste step is
where structure gets lost silently: a tool that ignores the closed-set
verdict convention, or leaves an empty table instead of a "none found"
row, only gets caught if a human notices on read. Wyatt exists so that
check happens at write time, not at review time, and so submitting
findings from any of the three tools is one tool call instead of a
file move.

## 2. What does it introduce or change?

An MCP server with four tools, callable directly by Claude, ChatGPT (as a
custom connector), and Perplexity (via BYOC):

- `create_brief(slug, title, questions[])` — writes `research/<slug>/topic.yaml`
  and `brief.md`, opens branch `research/<slug>`.
- `submit_findings(slug, source, content)` — `source ∈ {claude, chatgpt,
  perplexity, human}`. Validates: one section per brief question in order;
  verdict ∈ `{adopt/reference, differentiate, ignore}`; no empty tables
  (a null result needs a `none found` row with the reason in Notes). Passes
  → commits `research/<slug>/findings/<source>-findings.md`. Fails → returns
  the specific violation, writes nothing.
- `get_brief(slug)` / `get_status(slug)` — read the brief and see which
  sources have submitted, without re-pasting context into each tool.
- `publish(slug)` — opens a PR from `research/<slug>` to `main` if none
  exists; idempotent if one's already open.

Writes go through the GitHub Contents API (read current `sha`, write with
it), not a local clone — so the server stays stateless and concurrent
submissions from two tools at once fail as a retry-with-fresh-sha, not
data loss. The GitHub write token lives only in Wyatt's own environment;
none of the three client tools ever holds it directly.

This does **not** change how the git corpus itself gets merged — `publish`
opens a PR, it never merges one. That stays a human action, same principle
`docs/research/` conventions already assume elsewhere.

## 3. How is the result validated?

- A submission missing a brief question's section, using an invented
  verdict label, or containing an empty table with no "none found" row is
  rejected with the specific rule it broke — never partially committed.
- A valid submission from each of the three `source` values lands at the
  expected path with the expected filename convention
  (`<source>-findings.md`).
- Two submissions racing on the same file: the second attempt's stale
  `sha` is rejected by GitHub's API, not silently overwritten.
- `publish` called twice on the same slug doesn't open a second PR.

## 4. How would another engineer test it?

Spin up Wyatt locally against a scratch repo. Call `create_brief` with a
2-question brief. Call `submit_findings` three times with `source` set to
each of `claude`/`chatgpt`/`perplexity`, using one deliberately malformed
payload (missing a section) to confirm rejection, then a corrected one to
confirm it lands. Fire two `submit_findings` calls for the same `source`
concurrently against a stale `sha` to confirm the second fails cleanly
rather than clobbering the first. Call `publish` twice and confirm only
one PR exists on the second call.
