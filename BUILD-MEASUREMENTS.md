# Measurements

Everything on this page was **measured by the harness** that drove the build —
not by the agent. The agent's own account of what it did is in
[`BUILD-REPORT.md`](BUILD-REPORT.md) and the "How this was built" section of
[`README.md`](README.md).

- **Model**: `groq/openai/gpt-oss-120b` (a free model)
- **Agent**: XMINAL v0.28.0 — `xminal run`
- **Date**: 2026-11-15

## The three phases

| Phase | What it was | Mode | Turns | Tool calls | Agent time | Cost |
|---|---|---|---|---|---|---|
| **A** | plan the implementation (`PLAN.md` → `AGENT-PLAN.md`) | `plan` | 2 | 1 | **8.7 s** | $0.0046 |
| **B** | build the whole project | `build` | 40 | 73 | **6 m 14 s** | $0.0024 |
| **C** | finish the remaining steps + write the docs | `build` | 9 | 11 | **2 m 15 s** | $0.0000 |
| | **total** | | **51** | **85** | **8 m 38 s** | **$0.0070** |

Tokens: **642,405 in · 33,795 out** across the two build phases.

Phase B stopped at the 40-turn ceiling (`max_turns`) with the project working but
the documentation unwritten; phase C resumed the *same session* and finished it.
That is the honest shape of the run — not one clean sweep.

## Which features were exercised

102 tool calls were recorded (every call the agent made, with its duration and
its error flag):

| Tool | Calls | What it was used for here |
|---|---|---|
| `todo` | 25 | the build checklist, kept live across the whole run |
| `read` | 22 | the spec, the plan, its own files, the test output |
| `bash` | 17 | the test suite, the CLI, git, `md5sum` proofs |
| `write` | 11 | every source file, both documents, `.gitignore` |
| `edit` | 9 | the require-path fix, the test-path fix, the README section |
| `list` | 6 | surveying the project |
| `checkpoint` | 2 | snapshot → edit → undo → prove the bytes are back |
| `capabilities` | 1 | discovering its own tools |
| `skill` | 1 | loading `design-lab` |
| `glob` | 1 | finding the files |
| `task` | 1 | delegating `test/run.js` to a subagent |
| `verify` | 1 | `npm test` through the verify runner |
| `memory` | 1 | recording the renderer design decision |
| `mcp_list` / `mcp_call` | 1 + 1 | reading `README.md` **through an MCP server** |
| `worktree` | 1 | the `feature/docs-polish` worktree |
| `grep` | 1 | locating the front-matter parser |

Sixteen distinct tools, five errors — every one of them a failed attempt the
agent diagnosed and fixed (the `require` path, the `/tmp` permission, the
subagent that did not write its file, the accidental `.xminal` commit).

**Not used**: `multi_edit` (the agent preferred `edit`), and the dedicated
`git_commit`/`git_branch` tools (it ran git through `bash`, which is what the
permission model expects for shell work).

## Time inside the tools

45.9 s of the 8 m 38 s was spent **inside tools**; the rest was model time.
The single slowest call was `task` (the subagent) at 38.4 s — everything else
was under two seconds.

## The result

```
npm test → 42 passed, 0 failed
node src/cli.js site/index.md → a complete, self-contained HTML page
git log  → 4aee1a4 (build) · 06d432b (build docs + git hygiene)
```

Nothing here was written by hand: every source file, every document, both
commits and the bug fix came out of the agent's own tool calls.
