# CLAUDE.md — NeoVerse Employee Form

## Project Overview

Single-file React application for collecting NeoVerse employee data. The entire app lives in `index.html` — there is no build system, bundler, or package manager.

## Tech Stack

- **React 18.2.0** — loaded via CDN (UMD), no npm
- **Babel Standalone 7.23.2** — runtime JSX transformation in-browser
- **Google Fonts** — "Outfit" font family
- **Google Apps Script** — form submissions POST to a Google Sheets backend

## Project Structure

```
/
├── index.html    # Entire application (HTML, CSS, JSX)
└── README.md     # Brief project description
```

There is no `package.json`, no `node_modules`, no TypeScript, no test framework, and no linter/formatter config.

## Development

### Running Locally

Open `index.html` directly in a browser. No dev server is required.

### No Build Step

Babel transforms JSX at runtime in the browser via a `<script type="text/babel">` tag. There is nothing to compile or bundle.

### No Tests or Linting

No testing infrastructure or linting/formatting tools are configured.

## Architecture

### Components (defined in index.html)

- **`Section`** — Reusable wrapper for form sections (title, subtitle, colored header bar)
- **`Field`** — Reusable wrapper for form fields (label, required indicator, hint text)
- **`NeoVerseEmployeeForm`** — Main component; manages all form state and submission logic

### State Management

React `useState` hooks in the main component:
- `form` — object holding all form field values
- `showManager` — boolean toggle for confidential manager section
- `copied` — clipboard copy feedback
- `submitting` — loading state during submission
- `submitStatus` — outcome of submission (`success` / `error` / `missing` / `sent`)

### Key Functions

- `set(field)` — higher-order handler for updating individual form fields
- `toggleCredential` — array toggle for multi-select credential checkboxes
- `generateExport` — builds a plain-text export of the form data
- `handleCopy` — copies export text to clipboard
- `handleDownload` — triggers a `.txt` file download of export text
- `handleSubmitToNeoVerse` — POSTs form data to Google Apps Script endpoint

### Constants

Defined at the top of the script block:
- `SCRIPT_URL` — Google Apps Script endpoint
- `LOCATIONS` — 8 office/facility locations
- `BUSINESSES` — 4 business units
- `EMPLOYMENT_TYPES` — 4 employment categories
- `DEPARTMENTS` — 10 department options
- `CREDENTIAL_OPTIONS` — 23 professional credentials

## Code Conventions

- **Functional components only** — no class components
- **camelCase** for variables and functions
- **UPPER_SNAKE_CASE** for constants
- **Inline styles** — style objects (`inputStyle`, `textareaStyle`, `selectStyle`) instead of CSS classes
- **Manager-only fields** prefixed with `manager` (e.g., `managerPerformanceNotes`)
- **No external CSS framework** — all styling is hand-written with inline style objects and a `<style>` block

## Color Scheme

- Primary dark: `#0D2137`, `#163252`, `#1a4068`
- Accent gradient: `#2563eb` to `#7c3aed`
- Section header colors vary by category (blue, cyan, purple, gray, dark red)
- Text: white (`#fff`), light grays (`#e2e8f0`, `#94a3b8`)

## API Integration

Form data is submitted via `fetch` POST to a Google Apps Script URL. On network failure, a fallback request is sent with `mode: 'no-cors'`. Success/error feedback is shown to the user with auto-dismiss timers.
