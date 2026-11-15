---
title: Welcome to Quillmark
tags: markdown, static-site, demo
---

# Quillmark

A dependency-free Markdown → static-site generator.

## Features

- Headings, paragraphs and lists
- Fenced code blocks
- Inline formatting: bold, italic, code, links
- Self-contained HTML output with inline CSS

## Example code

```js
const { parse } = require('./src/markdown');
const { render } = require('./src/render');
const html = render(parse(markdown));
```

Visit the [project page](https://example.com) for more.
