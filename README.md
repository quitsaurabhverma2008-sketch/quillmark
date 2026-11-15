# Quillmark

A dependency-free Markdown → static-site generator. CommonJS, Node ≥ 18, no dependencies.

## Install

```sh
npm i -g .
# or run locally
node src/cli.js
```

## Usage

### CLI

```sh
node src/cli.js <input.md> [-o <out.html>] [--title <title>]
```

- `<input.md>` — markdown file to convert (required).
- `-o <out.html>` — write HTML to this file instead of stdout.
- `--title <title>` — override the page `<title>`.

Examples:

```sh
node src/cli.js site/index.md
node src/cli.js site/index.md -o site/index.html
node src/cli.js site/index.md --title "My Page" -o out.html
```

Exit codes: `0` on success, `1` on missing input file or unknown flag.

### API

```js
const markdown = require('./src/markdown');
const render = require('./src/render');

const { frontMatter, ast } = markdown.parse(fs.readFileSync('page.md', 'utf8'));
const html = render.render({ frontMatter, ast }, { title: 'My Page' });
```

## Project structure

```
src/
  markdown.js   # front-matter extraction + line-based AST parser
  render.js     # AST → HTML renderer with inline CSS
  cli.js        # command-line interface
test/
  run.js        # pure-Node test suite (no dependencies)
docs/
  USAGE.md      # detailed usage notes
site/
  index.md      # example markdown file (renderable via the CLI)
```

## Spec

See `PLAN.md` for the full project specification.

## How this was built

Numbered list of every step taken, with the capability used:

1. Read `PLAN.md` (spec) and `AGENT-PLAN.md` (implementation plan) — `read`
2. Listed available tools — `capabilities`
3. Loaded `design-lab` skill (checked applicability; not used for this CLI build) — `skill`
4. Surveyed the directory (only `PLAN.md` and `AGENT-PLAN.md` existed) — `glob`, `list`
5. Wrote `package.json` — `write`
6. Wrote `src/markdown.js` (front-matter extractor + line-based AST parser) — `write`
7. Wrote `src/render.js` (AST → HTML renderer with inline CSS) — `write`
8. Wrote `src/cli.js` (command-line interface) — `write`
9. Wrote `docs/USAGE.md` and `site/index.md` — `write`
10. Attempted to delegate `test/run.js` to a subagent — `task` (subagent did not create the file)
11. Wrote `test/run.js` myself (14 test blocks, 42 assertions) — `write`
12. Ran `node test/run.js` — `bash` → failed: `Cannot find module './src/markdown'`
13. **Bug found and fixed:** `src/render.js` and `src/cli.js` required `./src/markdown` from inside `src/`, resolving to `src/src/markdown`. Changed both to `require('./markdown')` — `edit`
14. Re-ran tests — `bash` → 40 passed, 1 failed; the failure was `/tmp` being read-only (EACCES), an environment issue, not a code bug
15. Fixed the test output path to use `os.homedir()` instead of `/tmp` — `edit`
16. Re-ran tests — `bash` → 42 passed, 0 failed
17. Confirmed with `npm test` — `verify` → green (42 passed, 0 failed)
18. Wrote `.gitignore` — `write`, then edited twice — `edit`
19. Wrote base `README.md` — `write`
20. Checkpoint/undo proof: snapshotted `src/markdown.js` — `checkpoint create`; made a temporary edit — `edit`; restored via `checkpoint undo`; verified md5sum returned to original `9658538aae160c0ba6bd32b451159fd0` — `bash`
21. Recorded renderer design decision (escapeHtml + whitelisted inline HTML) to project memory — `memory`
22. Read `README.md` through the configured `local` MCP server — `mcp_call`
23. `git init`, `git add -A`, committed everything as `4aee1a4` — `git` (accidentally committed the `.xminal` runtime folder)
24. Rendered `site/index.md` → `site/index.html` via the CLI — `bash` (step 1 of this request)
25. Located the front-matter parser — `grep` → `src/markdown.js:35` (step 2 of this request)
26. Added this "How this was built" section — `edit` (step 3 of this request)
27. Wrote `BUILD-REPORT.md` with the full replayable build log — `write` (step 4 of this request)
28. Untracked `.xminal`, added `.xminal-worktrees/` to `.gitignore`, committed — `git` (step 5 of this request)
