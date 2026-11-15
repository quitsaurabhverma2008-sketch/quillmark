turns: 2 | cost: $0.0046 | model: groq/openai/gpt-oss-120b
---
**Implementation Plan for Quillmark**

| Step | What to create | Core responsibilities |
|------|----------------|------------------------|
| 1️⃣ | **`src/markdown.js`** | • Read a markdown string (or file content). <br>• Detect optional front‑matter block (`---` … `---`). <br>• Parse the YAML‑ish key/value pairs (simple `key: value` lines – no nesting). <br>• Strip the front‑matter and produce a plain‑markdown body. <br>• Tokenise the body line‑by‑line into a *document tree* (AST) with the following node types: <br> – `heading` (level 1‑6) <br> – `paragraph` <br> – `ulist` / `olist` (array of `listItem`) <br> – `codeBlock` (fenced triple‑backticks) <br> – `inline` children for **bold**, *italic*, `` `code` ``, and `[text](url)` . |
| 2️⃣ | **`src/render.js`** | • Accept the AST and the front‑matter object. <br>• Walk the tree and emit a single HTML string. <br>• Insert a small, self‑contained `<style>` block (fonts, margins, code styling). <br>• Resolve the page `<title>`: use front‑matter `title` if present, otherwise the text of the first heading. |
| 3️⃣ | **`src/cli.js`** | • Minimal CLI parsing: `node src/cli.js <input.md> [-o <out.html>] [--title <title>]`. <br>• Validate that `<input.md>` exists → exit 1 on missing file. <br>• Read the file, call `markdown.parse`, then `render.render`. <br>• If `--title` is supplied, override the title in the rendered HTML. <br>• Print HTML to stdout **or** write to the file given by `-o`. <br>• Exit 0 on success. |
| 4️⃣ | **`package.json`** | • Define `"type": "commonjs"` (Node ≥ 18). <br>• `"bin": { "quillmark": "./src/cli.js" }`. <br>• `"scripts": { "test": "node test/run.js" }`. |
| 5️⃣ | **`test/run.js`** | • Pure‑Node test runner (no external libs). <br>• Use `assert` from Node. <br>• Create a set of inline markdown fixtures covering every supported feature (front‑matter, headings, lists, code block, inline formatting, link). <br>• For each fixture: <br> – Call `markdown.parse` → compare resulting AST shape with an expected object. <br> – Call `render.render` → compare the HTML string with an expected minimal HTML (ignoring whitespace differences). <br>• Integration test: spawn the CLI (`child_process.execFile`) with a sample file, verify stdout / file output and exit codes. |
| 6️⃣ | **`docs/USAGE.md`** | • Explain installation (`npm i -g .`), command‑line usage, API usage (`require("./src/markdown")` …). |
| 7️⃣ | **`site/index.md`** | • A simple example markdown file that the CLI can be run against. |
| 8️⃣ | **Optional housekeeping** | • Add a `.gitignore` (node_modules, *.log). <br>• Add a small README referencing the spec. |

---

### Risks & Mitigations  

| Risk | Why it matters | Mitigation |
|------|----------------|------------|
| **Front‑matter parsing** – malformed YAML (missing closing `---`, extra spaces) | Could cause parser to treat the whole file as front‑matter or crash. | Implement a tolerant line‑by‑line parser: stop at the next line that is exactly `---`. If not found, treat the whole file as body and ignore front‑matter. |
| **List detection** – mixing ordered and unordered markers, indented lists | Mis‑identifying list boundaries leads to wrong AST. | Use a simple state machine: a block of consecutive lines that start with `- ` → `ulist`; lines that start with `\d+. ` → `olist`. Break on blank line or a line not matching the current list type. |
| **Inline formatting nesting** (e.g., `**bold *italic*` ) | Full markdown nesting is complex; the spec only requires basic inline tags. | Implement a flat regex pass in order: code → bold → italic → link → plain text. Nested cases will be rendered in a reasonable, if not perfectly spec‑compliant, way. |
| **HTML escaping** – raw `<` or `&` in text | Could produce malformed HTML. | Escape `&`, `<`, `>` in plain text nodes before inserting into HTML. |
| **CLI argument edge‑cases** – missing `-o` value, unknown flags | May cause unexpected crashes. | Use a minimal argument parser that validates flag arguments; on unknown flags print usage and exit 1. |
| **Cross‑platform line endings** (`\r\n` vs `\n`) | May break front‑matter detection. | Normalise file content to `\n` before processing. |
| **Testing without dependencies** – need to compare HTML while ignoring whitespace | Direct string equality can be flaky. | Normalise both expected and actual HTML with a simple function that collapses consecutive whitespace and trims. |

---

### How to Test

1. **Unit tests** (inside `test/run.js`) for each markdown feature.  
2. **AST shape tests** – compare deep equality of parsed tree with a hand‑crafted expected object.  
3. **HTML output tests** – compare rendered string after normalising whitespace.  
4. **CLI tests** – spawn the script with valid and invalid inputs, assert: <br> • stdout contains HTML, <br> • exit code is `0` on success, `1` on missing file, <br> • `-o` creates the file with the same content as stdout.  
5. Run `npm test` and verify that the process exits with status 0 only when all assertions pass.

---

## 📋 Implementation Checklist  

1. Create `src/markdown.js` with front‑matter extraction and a line‑based parser that builds the AST.  
2. Create `src/render.js` that walks the AST and returns a full HTML string with inline CSS and proper `<title>`.  
3. Create `src/cli.js` that parses arguments, reads the input file, calls the parser & renderer, handles `-o` and `--title`, and exits with appropriate codes.  
4. Add `package.json` containing `bin`, `scripts.test`, and `"type": "commonjs"`.  
5. Write `test/run.js` – set up fixtures, assert parser output, renderer output, and CLI behaviour using Node’s `assert` and `child_process`.  
6. Draft `docs/USAGE.md` with install, CLI, and API usage examples.  
7. Add a sample markdown file at `site/index.md`.  
8. (Optional) Add `.gitignore` and a brief `README.md`.  
9. Run `npm test`; fix any failing assertions.  
10. Verify the CLI works manually: `node src/cli.js site/index.md -o out.html`.  

Following this plan will deliver a minimal, dependency‑free Quillmark generator that satisfies the spec while keeping the change surface small and well‑tested.
