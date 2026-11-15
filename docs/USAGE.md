# Quillmark — usage

A dependency-free Markdown → static-site generator. CommonJS, Node ≥ 18, no
dependencies, no CDN, no build tooling.

## Install

```sh
npm i -g .
```

This installs the `quillmark` binary (see `bin` in `package.json`). You can also
run the CLI directly with `node src/cli.js`.

## CLI

```
node src/cli.js <input.md> [-o <out.html>] [--title <title>]
```

- `<input.md>` — markdown file to convert (required).
- `-o <out.html>` — write the rendered HTML to this file instead of stdout.
- `--title <title>` — override the page `<title>` (default: front-matter `title`,
  then the first heading).

Exit codes: `0` on success, `1` when the input file is missing or arguments are
invalid.

Examples:

```sh
# Print HTML to stdout
node src/cli.js site/index.md

# Write to a file
node src/cli.js site/index.md -o site/index.html

# Override the title
node src/cli.js site/index.md --title "My Page"
```

## Module API

```js
const { parse } = require('./src/markdown');
const { render } = require('./src/render');

const { frontMatter, ast } = parse(fs.readFileSync('page.md', 'utf8'));
const html = render({ frontMatter, ast }, { title: 'Custom title' });
```

`parse` returns `{ frontMatter, ast }`. The AST uses node types `heading`,
`paragraph`, `ulist`, `olist`, `codeBlock` with inline children `text`, `bold`,
`italic`, `code`, `link`.

`render` returns a single self-contained HTML string with inline CSS.

## Markdown features

- Front matter: `---` fenced `title:` and `tags:` at the top of the file.
- Headings: `#` through `######`.
- Paragraphs.
- Unordered lists: `- item`.
- Ordered lists: `1. item`.
- Fenced code blocks: ```.
- Inline code: `` `code` ``.
- Bold: `**bold**`.
- Italic: `*italic*`.
- Links: `[text](url)`.

## Rendering

Each document becomes one self-contained HTML file: inline `<style>`, no
external assets, `<title>` from the front matter or the first heading.
