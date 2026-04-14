# CLAUDE.md — Severino dos Contratos

## Project Overview

**Severino dos Contratos** is a browser-based legal contract generator for Brazilian real estate transactions. It is a **single-file, zero-dependency web application** written in vanilla HTML/CSS/JavaScript (Portuguese, pt-BR).

The entire application lives in `index.html` (≈836 lines). There is no build step, no package manager, and no backend.

---

## Repository Layout

```
severino/
├── index.html      # Complete application (HTML + embedded CSS + embedded JS)
├── CLAUDE.md       # This file
└── cep.html        # Standalone CEP (ZIP code) lookup tool for iPhone
```

---

## Technology Stack

| Layer        | Choice                                      |
|--------------|---------------------------------------------|
| Language     | Vanilla JavaScript (ES6+), HTML5, CSS3      |
| Framework    | None                                        |
| Libraries    | None — ZIP/DOCX generation is hand-rolled   |
| Package mgr  | None                                        |
| Build tool   | None                                        |
| Backend      | None — fully client-side                    |
| Locale       | Brazilian Portuguese (pt-BR)                |

---

## Architecture of `index.html`

The file is structured in three sequential sections:

### 1. Embedded CSS (lines 7–139)
- Color palette: dark brown `#1c0f00`, mid brown `#6b3010`, gold `#c9922a`, cream `#f5ede0`
- Layout: sticky header (72px) + CSS Grid sidebar (340px fixed) + fluid main area
- Components: header, tabs, cards, modals, buttons, scrollbar customization

### 2. HTML Structure (lines 141–388)
- **Header**: inline SVG logo ("Severino" caricature), tab buttons
- **Editor tab**: sidebar form (contract type, parties, variables) + main preview area
- **Banco de Cláusulas tab**: clause database management UI
- **Modal**: create/edit individual clauses

### 3. JavaScript Logic (lines 390–834)

Key data structures:
```js
clauses[]        // Array of clause objects {id, section, title, text, tags[]}
selected         // Set<id> of currently selected clause IDs
vendedores[]     // Array of seller person objects
compradores[]    // Array of buyer person objects
customVars{}     // Map of user-defined template variables
```

Key functions:

| Function          | Purpose                                                  |
|-------------------|----------------------------------------------------------|
| `gerarDocx()`     | Entry point — builds and triggers download of .docx file |
| `buildZip()`      | Constructs a ZIP binary from scratch (no lib)            |
| `renderTpl(str)`  | Replaces `{{variable}}` tokens with runtime values        |
| `buildBloco()`    | Generates legal qualification paragraphs for parties     |
| `updatePreview()` | Live-renders the contract in the preview pane            |
| `renderBanco()`   | Renders the clause database view                         |

---

## Contract Types

The app supports four contract types (stored in the `SECTIONS` constant):

1. Compra e Venda de Imóvel Urbano
2. Compromisso de Compra e Venda de Imóvel Urbano
3. Compra e Venda de Lote Urbano
4. Compra e Venda com Dação em Pagamento variants

---

## Clause Sections (9 categories)

`Qualificação` → `Objeto` → `Preço e Pagamento` → `Posse e Entrega` → `Tributos` → `Garantias` → `Mora e Inadimplemento` → `Intermediação` → `Disposições Finais`

---

## Template Variable System

Clauses use `{{token}}` placeholders. Available tokens include:

- Party data: `{{vendedor}}`, `{{comprador}}`, `{{cpf_vendedor}}`, etc.
- Property: `{{imovel}}`, `{{area}}`, `{{matricula}}`, `{{cartorio}}`
- Financial: `{{valor}}`, `{{sinal}}`, `{{parcelas}}`
- Custom: any key added via the `customVars` interface

The `renderTpl()` function handles substitution. Portuguese gender/number agreement (masculine/feminine/plural pronouns) is handled by inspecting party metadata and applying pronoun rules at render time.

---

## DOCX Generation (no external library)

`gerarDocx()` → `buildZip()` creates a valid Office Open XML (`.docx`) ZIP archive entirely in JavaScript using:
- Manual CRC-32 calculation
- Deflate-encoded XML parts
- Proper ZIP local/central directory headers

The output is a Blob URL triggered via `<a download>`.

---

## Code Conventions

- **Naming**: camelCase for functions/variables; UPPER_SNAKE_CASE for top-level constants; kebab-case for HTML `id` attributes
- **Language**: all identifiers, UI strings, comments, and user-facing text are in Brazilian Portuguese
- **DOM**: manipulation via `innerHTML` / `insertAdjacentHTML`; event binding via inline `onclick` attributes
- **No classes**: the codebase uses plain functions and module-level variables
- **No modules**: everything is in one `<script>` block; no `import`/`export`
- **Iteration**: prefer `Array.map/filter/forEach/find` over `for` loops

---

## Development Workflow

Since there is no build step, the development cycle is:

1. Edit `index.html` directly in any text editor
2. Open (or refresh) the file in a browser — `file://` protocol works fine
3. Use browser DevTools for debugging
4. No compilation, transpilation, or bundling required

### Testing

There is no automated test suite. Manual testing covers:
- Adding sellers/buyers and confirming pronoun conjugation
- Selecting clauses from multiple sections and previewing the result
- Generating a `.docx` file and opening it in Word/LibreOffice
- Creating, editing, and deleting custom clauses in the Banco tab

---

## Deployment

Any static file host works (Netlify, GitHub Pages, Vercel, S3, or simply opening the file locally). No server-side component exists.

---

## Author / Contact

Adriano Scomparin · adrianoscomparin@gmail.com · (66) 99955-2744 · Sinop/MT

---

## AI Assistant Guidelines

- **Do not introduce dependencies** — keep the app zero-dependency unless the user explicitly requests an external library.
- **Preserve the single-file pattern** — all new features should be embedded in `index.html` unless a new standalone tool is explicitly requested (like `cep.html`).
- **Match existing conventions** — use Portuguese variable names, camelCase, inline event handlers, and `innerHTML` manipulation consistent with existing code.
- **No build tooling** — do not add webpack, Vite, TypeScript, or any transpiler without explicit user approval.
- **Test DOCX output** — any change to `gerarDocx()` or `buildZip()` must be verified by opening the generated file in a real Word processor.
- **Respect pronoun logic** — the gender/number conjugation system is subtle; read and understand it fully before modifying party-rendering code.
- **Branch**: develop on `claude/add-claude-documentation-Cw1XY`, push to `adrianoscomparin-boop/severino`.
