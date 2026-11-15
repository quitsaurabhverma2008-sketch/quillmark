'use strict';

// Quillmark markdown parser.
// Dependency-free, CommonJS, Node >= 18.
//
// Public API:
//   const { parse } = require('./src/markdown');
//   const { frontMatter, ast } = parse(markdownString);
//
// AST node types:
//   { type: 'heading', level: 1..6, children: [inline, ...] }
//   { type: 'paragraph', children: [inline, ...] }
//   { type: 'ulist', items: [ { type: 'listItem', children: [inline, ...] }, ... ] }
//   { type: 'olist', items: [ { type: 'listItem', children: [inline, ...] }, ... ] }
//   { type: 'codeBlock', lang: string, code: string }
//
// Inline node types:
//   { type: 'text', content: string }
//   { type: 'bold', content: string }
//   { type: 'italic', content: string }
//   { type: 'code', content: string }
//   { type: 'link', content: string, href: string }

function escapeHtml(text) {
  return String(text)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;');
}

// Parse an optional `---` fenced YAML-ish front-matter block.
// Returns { frontMatter, body } where body is the markdown with the block stripped.
// Tolerant: if there is an opening `---` but no closing `---`, the whole file is
// treated as body and no front-matter is extracted.
function parseFrontMatter(content) {
  const lines = content.split('\n');
  let frontMatter = {};
  let bodyStart = 0;

  if (lines.length > 0 && lines[0].trim() === '---') {
    let closeIdx = -1;
    for (let i = 1; i < lines.length; i++) {
      if (lines[i].trim() === '---') {
        closeIdx = i;
        break;
      }
    }
    if (closeIdx !== -1) {
      for (let i = 1; i < closeIdx; i++) {
        const m = lines[i].match(/^([A-Za-z_][A-Za-z0-9_]*)\s*:\s*(.*)$/);
        if (m) {
          const key = m[1];
          let value = m[2].replace(/\s+$/, '');
          if (key === 'tags') {
            frontMatter[key] = value.split(',').map(s => s.trim()).filter(Boolean);
          } else {
            frontMatter[key] = value;
          }
        }
      }
      bodyStart = closeIdx + 1;
    }
    // No closing fence -> treat entire file as body (frontMatter stays {}).
  }

  const body = lines.slice(bodyStart).join('\n');
  return { frontMatter, body };
}

// Parse inline markdown into a flat array of inline nodes.
// Order of precedence: code -> bold -> italic -> link -> plain text.
// Nesting is not fully supported; nested cases render in a reasonable way.
function parseInline(text) {
  const children = [];
  let i = 0;
  const n = text.length;
  while (i < n) {
    const rest = text.slice(i);
    let m;

    // Inline code: `code`
    if ((m = rest.match(/^(`)([^`]+?)\1/))) {
      children.push({ type: 'code', content: m[2] });
      i += m[0].length;
      continue;
    }
    // Bold: **bold**
    if ((m = rest.match(/^(\*\*)([^*]+?)\1/))) {
      children.push({ type: 'bold', content: m[2] });
      i += m[0].length;
      continue;
    }
    // Italic: *italic*
    if ((m = rest.match(/^(\*)([^*]+?)\1/))) {
      children.push({ type: 'italic', content: m[2] });
      i += m[0].length;
      continue;
    }
    // Link: [text](url)
    if ((m = rest.match(/^(\[)([^\]]+)\]\(([^)]*)\)/))) {
      children.push({ type: 'link', content: m[2], href: m[3] });
      i += m[0].length;
      continue;
    }

    // Plain text: accumulate until the next inline marker character.
    let next = n;
    for (const marker of ['`', '*', '[']) {
      const idx = rest.indexOf(marker);
      if (idx !== -1 && idx < next) next = idx;
    }
    if (next === 0) {
      // A marker character that did not form a valid inline construct.
      children.push({ type: 'text', content: rest[0] });
      i += 1;
    } else {
      children.push({ type: 'text', content: rest.slice(0, next) });
      i += next;
    }
  }
  return children;
}

// Parse a markdown string into { frontMatter, ast }.
function parse(content) {
  const normalized = content.replace(/\r\n/g, '\n').replace(/\r/g, '\n');
  const { frontMatter, body } = parseFrontMatter(normalized);
  const lines = body.split('\n');
  const ast = [];
  let i = 0;
  const n = lines.length;

  while (i < n) {
    const line = lines[i];

    // Blank lines separate blocks; skip them.
    if (line.trim() === '') {
      i += 1;
      continue;
    }

    // Fenced code block: ```
    if (line.match(/^```/)) {
      const fenceMatch = line.match(/^```(\w*)/);
      const lang = fenceMatch ? fenceMatch[1] : '';
      const codeLines = [];
      i += 1;
      while (i < n && !lines[i].match(/^```/)) {
        codeLines.push(lines[i]);
        i += 1;
      }
      if (i < n) i += 1; // consume closing fence
      ast.push({ type: 'codeBlock', lang, code: codeLines.join('\n') });
      continue;
    }

    // Heading: # .. ######
    const h = line.match(/^(#{1,6})\s+(.*)$/);
    if (h) {
      ast.push({ type: 'heading', level: h[1].length, children: parseInline(h[2]) });
      i += 1;
      continue;
    }

    // Unordered list: - item
    if (line.match(/^- /)) {
      const items = [];
      while (i < n && lines[i].match(/^- /)) {
        items.push({ type: 'listItem', children: parseInline(lines[i].slice(2)) });
        i += 1;
      }
      ast.push({ type: 'ulist', items });
      continue;
    }

    // Ordered list: 1. item
    if (line.match(/^\d+\. /)) {
      const items = [];
      while (i < n && lines[i].match(/^\d+\. /)) {
        items.push({ type: 'listItem', children: parseInline(lines[i].replace(/^\d+\. /, '')) });
        i += 1;
      }
      ast.push({ type: 'olist', items });
      continue;
    }

    // Paragraph: consecutive non-blank lines that are not a block start.
    const paraLines = [];
    while (
      i < n &&
      lines[i].trim() !== '' &&
      !lines[i].match(/^```/) &&
      !lines[i].match(/^#{1,6}\s+/) &&
      !lines[i].match(/^- /) &&
      !lines[i].match(/^\d+\. /)
    ) {
      paraLines.push(lines[i]);
      i += 1;
    }
    ast.push({ type: 'paragraph', children: parseInline(paraLines.join(' ')) });
  }

  return { frontMatter, ast };
}

module.exports = { parse, escapeHtml };
