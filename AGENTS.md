# AGENTS.md

Instructions for AI coding agents working on **xlsx-write-stream** (`xlsx-stream` on GitHub).

## Project overview

Streaming XLSX writer for Node.js. Replaces CSV for large Excel exports with low memory usage. Built on Node.js `Transform` streams and `archiver` (ZIP/OOXML).

**Scope:** CSV replacement only. Do not add formatting, charts, comments, or other OOXML features unless explicitly requested.

**Supported cell value types:** `string`, `number`, `Date`, `boolean` (coerced to string in XML).

## Repository layout

```
src/
  index.js                 # CommonJS entry; re-exports default from XLSXTransformStream
  XLSXTransformStream.js   # Main Transform stream; assembles the XLSX ZIP archive
  XLSXRowTransform.js      # Row-level Transform; emits sheet XML header/rows/footer
  utils.js                 # Cell ID, sanitization, number/date format helpers
  templates/               # Static OOXML fragments and row/cell builders
test/                      # Mocha specs (*.spec.js); snapshot XLSX in test/test.xlsx
dist/                      # TypeScript build output (published to npm); do not edit by hand
```

## Commands

Run these before considering work complete:

```bash
npm install          # Install dependencies
npm run build        # Compile src/ → dist/ via tsc (runs clean first)
npm test             # build + mocha with babel-core/register
npm run lint         # eslint src test
```

Optional:

```bash
npm run test-cov     # Coverage via isparta
npm run clean        # Remove dist/
```

CI runs `npm test` on Node.js 16, 18, and 20, then `npm run lint` on Node 20.

## Code style

Match existing patterns in `src/` and `test/`.

| Topic | Convention |
|-------|------------|
| Indentation | 4 spaces (see `.editorconfig`) |
| Line endings | LF, UTF-8, final newline |
| Lint | ESLint `airbnb-base` (`.eslintrc`); max line length 150 |
| Modules | ES6 `import`/`export` in `src/`; `export default` for main classes |
| Strict mode | `"use strict"` enforced globally by ESLint |
| Classes | JSDoc on public classes and constructor options |
| Naming | PascalCase classes (`XLSXTransformStream`), camelCase functions/variables |
| Templates | Pure functions in `src/templates/`; export via `templates/index.js` barrel |

Example class pattern:

```javascript
/** Class representing a XLSX Transform Stream */
export default class XLSXTransformStream extends Transform {
    /**
     * @param options {Object}
     * @param options.shouldFormat {Boolean}
     */
    constructor(options = {}) {
        super({ objectMode: true });
        // ...
    }
}
```

## Architecture notes

- **Input contract:** Readable stream in `objectMode`; each chunk is an `Array` of cell values.
- **Pipeline:** `XLSXTransformStream` wraps `archiver` and pipes `XLSXRowTransform` into `xl/worksheets/sheet1.xml`.
- **Backpressure:** `_transform` respects `rowTransform.write()` drain events.
- **XML safety:** User strings go through `sanitize()` in `utils.js`; do not bypass it for cell content.
- **Build:** Source is JavaScript with JSDoc types; `tsc` emits `dist/` and `.d.ts` typings (`package.json` → `"typings": "dist/index.d.ts"`).

## Testing

- Framework: Mocha + Chai (+ Sinon where needed).
- Test files: `test/**/*.spec.js`, named after the module under test.
- Tests require Babel register (`babel-core/register`); follow existing `import` style in specs.
- Integration tests unzip generated XLSX and compare against `test/test.xlsx` snapshot bytes.
- Add or update tests for behavior changes; run `npm test` locally.

## Boundaries

### Always

- Keep changes minimal and focused on the requested task.
- Run `npm test` and `npm run lint` after code changes.
- Update `CHANGELOG.md` for user-visible changes (follow existing version/date format).
- Preserve streaming semantics and low memory usage.

### Ask first

- Adding or upgrading major dependencies.
- Changing the public API or default export shape.
- Breaking changes to input/output stream contracts.

### Never

- Commit `node_modules/` or edit `dist/` manually (regenerate via `npm run build`).
- Add OOXML features outside the CSV-replacement scope without approval.
- Push directly to `master` expecting a stable release (see release process below).

## Git and release

- Default branch: `master`.
- Pull requests trigger the **Check** workflow (build, test, lint).
- Pushing to `master` publishes a **beta** npm version automatically.
- **Latest** npm releases are cut manually via GitHub Releases.
- Add `[skip ci]` to a commit message to skip CI/release.
- See `CONTRIBUTING.md` for release details.

## Tool-specific rules

Cursor loads additional rules from `.cursor/rules/`. Those files extend this document with scoped, enforceable conventions. When in doubt, follow the nearest `AGENTS.md` and applicable Cursor rules.
