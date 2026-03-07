# CLAUDE.md — NeoVerse Employee Form

## Quick Facts

- **Single-file app** — everything is in `index.html` (~402 lines). No other source files.
- **No build system** — no `package.json`, no bundler, no TypeScript. Open `index.html` in a browser to run.
- **No tests or linting** — nothing to run before committing.
- **React via CDN** — React 18.2.0 + ReactDOM loaded as UMD scripts, Babel Standalone transforms JSX at runtime.
- **Backend** — form data POSTs to a Google Apps Script endpoint that writes to Google Sheets.

## Business Context

NeoVerse is a physical therapy / performance / golf enterprise on the Mississippi Gulf Coast. This form collects employee profile data across their business units (NeoLife PT, NeoFit Performance, NeoGolf, NeoVerse Enterprise). Managers Robby & Rebecca use a hidden confidential section for performance/compensation notes.

## File Layout

```
index.html        ← entire app: HTML head, <style>, <script type="text/babel"> with all React code
README.md         ← two-line description
CLAUDE.md         ← this file
```

### index.html Structure (line references)

| Lines | Content |
|-------|---------|
| 1–15 | HTML head: CDN scripts (React, ReactDOM, Babel), Google Fonts, global CSS reset |
| 18–26 | Constants: `SCRIPT_URL`, `LOCATIONS`, `BUSINESSES`, `EMPLOYMENT_TYPES`, `DEPARTMENTS`, `CREDENTIAL_OPTIONS` |
| 28–40 | `defaultForm` object — all form field names with defaults |
| 42–60 | `Section` component — colored header bar + white content card |
| 62–75 | `Field` component — label, required asterisk, hint, children |
| 77–83 | Shared style objects: `inputStyle`, `textareaStyle`, `selectStyle` |
| 85–397 | `NeoVerseEmployeeForm` — main component (state, handlers, full JSX) |
| 399 | `ReactDOM.createRoot` render call |

## Architecture

### Components

- **`Section({ title, subtitle, children, color })`** — card wrapper with colored header. Each form section uses a different `color`.
- **`Field({ label, required, children, hint })`** — form field wrapper. Shows red asterisk when `required` is truthy.
- **`NeoVerseEmployeeForm`** — the entire form. All state lives here via `useState`.

### State Variables

| Variable | Type | Purpose |
|----------|------|---------|
| `form` | object | All form field values (see `defaultForm` for shape) |
| `showManager` | boolean | Toggles the confidential manager section |
| `copied` | boolean | Brief "Copied!" feedback after clipboard copy |
| `submitting` | boolean | Loading state during POST |
| `submitStatus` | `null\|"success"\|"error"\|"missing"\|"sent"` | Submission outcome, auto-clears after 3–4s |

### Form Sections (in render order)

1. **Identity** (default color) — name, credentials, email, phone, emergency contact
2. **Employment** (`#1e40af`) — title, department, business, location, type, dates, reporting
3. **Clinical & Professional** (`#0e7490`) — specialties, patient populations, equipment
4. **Growth & Development** (`#7c3aed`) — strengths, goals, continuing education
5. **Personal** (`#475569`) — interests, additional notes
6. **Manager Notes** (`#991b1b`, hidden by default) — confidential performance/compensation notes

### Required Fields

Only these fields are marked required in the UI: `fullName`, `title`, `department`, `business`, `location`. However, only `fullName` is validated before submission (line 182).

### Data Export

Three output methods:
- **Submit to NeoVerse** — POST JSON to Google Apps Script (`SCRIPT_URL`)
- **Copy to Clipboard** — plain-text formatted via `generateExport()`
- **Download as .txt** — same text, saved as `NeoVerse_Profile_{name}.txt`

## Code Conventions

- **Functional components only**, no class components
- **camelCase** for variables/functions, **UPPER_SNAKE_CASE** for constants
- **Inline styles everywhere** — shared base styles in `inputStyle`/`textareaStyle`/`selectStyle` objects, everything else inline in JSX
- **Manager fields** prefixed with `manager` (e.g., `managerPerformanceNotes`)
- **Grid layouts** — two-column grids via `gridTemplateColumns: "1fr 1fr"` with `gap: 16`
- **Font** — "Outfit" at various weights (300–800)

## Common Modifications

### Adding a new form field

1. Add the field key with default value to `defaultForm` (line ~28)
2. Add the input JSX inside the appropriate `<Section>`, wrapped in a `<Field>` component
3. Use `onChange={set("fieldName")}` for simple text/select fields
4. If it should appear in exports, add a line in `generateExport()` (line ~103)
5. The Google Sheets script must also be updated to accept the new field

### Adding a new dropdown option

Add the value to the relevant constant array (`LOCATIONS`, `BUSINESSES`, `EMPLOYMENT_TYPES`, `DEPARTMENTS`, or `CREDENTIAL_OPTIONS`) at line ~22–26.

### Adding a new form section

Follow the pattern of existing sections:
```jsx
<Section title="Section Name" subtitle="Description" color="#hexcolor">
  <Field label="Field Label" hint="Help text">
    <input style={inputStyle} value={form.fieldName} onChange={set("fieldName")} placeholder="..." />
  </Field>
</Section>
```

### Changing styles

- Global reset is in the `<style>` tag (lines 11–14)
- Input/textarea/select base styles: `inputStyle`, `textareaStyle`, `selectStyle` (lines 77–83)
- Background gradient: line 236
- Section header colors: passed as `color` prop to each `<Section>`

## Gotchas

- **Single file** — all HTML, CSS, and JS/JSX live in one file. Don't try to import modules or split into separate files without also adding a bundler.
- **No module system** — React and ReactDOM are globals (`React.useState`, `ReactDOM.createRoot`). No `import` statements.
- **Babel in browser** — the `<script type="text/babel">` tag is transformed at runtime. This is slow for production but fine for this internal tool.
- **CORS fallback** — `handleSubmitToNeoVerse` catches fetch errors and retries with `mode: "no-cors"`, which means the response is opaque. The status shows "Sent" (not "Saved") in this case because success can't be confirmed.
- **Credentials field is an array** — unlike all other fields (strings), `form.credentials` is an array managed by `toggleCredential()`, not `set()`.
- **No form validation** — only `fullName` is checked before submission. All other "required" markers are visual-only.
