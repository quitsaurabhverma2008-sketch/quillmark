# Quillmark — the spec

A dependency-free Markdown → static-site generator. CommonJS, Node ≥ 18, no
dependencies, no CDN, no build tooling. Everything is plain files.

## The shape

```
package.json          "test": "node test/run.js", bin -> src/cli.js
src/markdown.js       parses front-matter + markdown into a document tree
src/render.js         renders a document tree into one HTML page
src/cli.js            the CLI: quillmark <input.md> [-o out.html] [--title T]
test/run.js           the test runner + assertions (no dependencies)
docs/USAGE.md         how to use the CLI and the module API
site/index.md         a sample page the CLI renders into site/index.html
```

## The rules

1. **Front matter** — a document may start with `---` fenced YAML-ish metadata
   (`title:`, `tags:`). The parser reads it, strips it, and keeps the pairs.
2. **Markdown** — headings (`#`..`######`), paragraphs, unordered lists
   (`- item`), ordered lists (`1. item`), fenced code blocks (```), inline code
   (`` `code` ``), bold (`**x**`), italic (`*x*`) and links (`[text](url)`).
3. **Rendering** — one self-contained HTML file per document: inline CSS, no
   external assets, `<title>` from the front matter or the first heading.
4. **The CLI** — `node src/cli.js <input.md> [-o <out.html>] [--title <title>]`
   prints the rendered HTML (or writes it to the file), exits 0 on success and
   non-zero when the input file is missing.
5. **Tests** — `npm test` runs `node test/run.js`, which asserts the parser and
   the renderer and exits non-zero on the first failure.
