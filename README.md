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
