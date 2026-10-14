# Code Style — Calculator System Front End (Student ID 832402111)

This document is the coding standard for the `832402111_calculator_frontend` repository. It describes
the rules the existing files already follow, so that a reviewer can check the code against a stated
standard and so that any later change stays consistent with it.

---

## 1. Source of the standard

The standard is based on published style guides, in this order of authority:

| Role | Reference |
| --- | --- |
| Primary basis — JavaScript | *Google JavaScript Style Guide* — <https://google.github.io/styleguide/jsguide.html> |
| Primary basis — HTML and CSS | *Google HTML/CSS Style Guide* — <https://google.github.io/styleguide/htmlcssguide.html> |
| Supplement — JavaScript | *Airbnb JavaScript Style Guide* — <https://github.com/airbnb/javascript> |
| Supplement — web platform conventions | *MDN Web Docs* — <https://developer.mozilla.org/> |

### 1.1 Taken from the Google JavaScript Style Guide

- **Naming.** `lowerCamelCase` for variables, functions and object properties; `CONSTANT_CASE` for module
  level constants that are never reassigned; descriptive names instead of abbreviations; predicate-style
  names for booleans (`active`, `busy`, `loading`).
- **Declarations and scope.** One declaration per statement, no implicit globals, every module in strict
  mode, dependencies declared before use, no reliance on hoisting for readability.
- **Formatting.** Two-space indentation, braces on the same line as the statement, no line longer than the
  agreed limit, a blank line between logical blocks, no trailing whitespace.
- **Documentation.** JSDoc with `@param` and `@returns` on functions that are more than a few lines or
  that cross a module boundary; a file-level comment that states the responsibility of the file.
- **Strings.** Single quotes, no unnecessary concatenation, no `eval`, no `Function` constructor.
- **Error handling.** Exceptions are caught where they can be handled; a failure that the user must know
  about is reported, not swallowed silently.

### 1.2 Taken from the Google HTML/CSS Style Guide

- Valid, semantic HTML5 with the `lang` attribute, `UTF-8` declared as the character set and the viewport
  meta tag present.
- Lower-case element and attribute names, double quotes around attribute values, every attribute value
  quoted.
- Separation of concerns: no `style` attribute for layout, no presentational markup, no inline `<script>`
  block with application logic.
- CSS: lower-case selectors and properties, one declaration per line, a semicolon after every
  declaration, a space after the colon, short hexadecimal colour notation where possible, class names that
  describe meaning rather than appearance.
- Stylesheets split into commented sections with a stated order.

### 1.3 Taken from the Airbnb guide and MDN

- **Airbnb:** prefer `const` over `let` and never use `var`, use `===` instead of `==`, use template
  literals only where interpolation is really needed, early `return` instead of deep nesting, name event
  handlers after the event they handle, keep functions small and single-purpose.
- **MDN:** use `textContent` where possible and never inject unescaped data into the document, prefer
  `addEventListener` over inline handlers, use `classList` instead of manipulating `className` strings,
  use `dataset` for `data-*` values, feature-detect APIs that are not universally available.

### 1.4 Where this project deliberately differs

| Google / Airbnb rule | What this project does instead | Why |
| --- | --- | --- |
| ES modules (`import` / `export`) with one module per file | Classic `<script>` tags, one IIFE per file, one object published on `window` per module | The assignment requires a folder that runs with **no build step** and no bundler. ES modules would need a module server or a bundling step, and `file://` loading would break. |
| `const` / `let` only, `var` forbidden | `const` and `let` in all new code; the existing files still contain `var` inside their IIFEs | The files were written with function-scoped declarations and are kept as they are rather than rewritten for cosmetics. New or edited code must use `const` and `let`; introducing a new `var` is not allowed. |
| 80-column line limit | **110-column limit** | The modules are heavily documented and the DOM lookups are long; 80 columns produced unreadable wrap-arounds, while 110 still fits a normal editor next to a diff view. |
| JSDoc on every function, including private ones | JSDoc on every **exported** function and on every function whose contract is not obvious | Keeps the useful documentation and avoids noise on two-line helpers. |
| Type annotations through the Closure compiler | JSDoc types for documentation only (`{string}`, `{object}`, `{Promise<...>}`) | No type checker is part of the project; the annotations are read by humans and by editors. |
| Google's own `gts` toolchain / ESLint config | Manual review plus `node --check` (see section 14) | The project has no `package.json` and therefore no dependency at all; a toolchain would contradict the "review it without installing anything" goal. |

---

## 2. File naming and repository layout

- **All file and folder names are lower-case.** No spaces, no upper-case letters, no mixed case.
- **One module per file.** A file contains exactly one IIFE and publishes exactly one global object; the
  file name and the object name correspond (`calculator.js` → `CalculatorPanel`, `api.js` →
  `CalculatorApi`).
- **Multi-word names use `kebab-case`.** This applies to any future multi-word CSS file such as
  `style-dark.css` or `print-layout.css`; the current stylesheet is a single `style.css`.
- **`index.html` is the entry point** and is the only HTML file. There are no per-feature pages: the three
  modes are tabs inside the one page.
- Directories: `src/` is the deployable unit, `src/css/` holds stylesheets, `src/js/` holds scripts. No
  file is placed in the repository root except `README.md` and this document.
- Assets that are not code (screenshots, submission material) do not live in `src/`, so that a copy of
  `src/` into a web root never ships documentation or images.

## 3. Module pattern

Every JavaScript file has the same shape:

```javascript
/**
 * Module header: what this file is responsible for, and what it must not do.
 *
 * Loaded order and dependencies are stated here when they are not obvious.
 */
(function exposeModule(global) {
  'use strict';

  // Dependencies are read from `window` once, at the top of the IIFE.
  var ui = global.CalculatorUi;
  var api = global.CalculatorApi;
  var config = global.CALCULATOR_CONFIG;

  /** Module constants in UPPER_SNAKE_CASE. */
  var HEALTH_INTERVAL = 15000;

  // ... private functions ...

  global.CalculatorSomething = {
    init: init,
    // public surface only
  };
})(window);
```

Rules:

1. **One global namespace object per module.** The published names are `window.CALCULATOR_CONFIG`
   (data only), `window.CalculatorApi`, `window.CalculatorUi`, `window.CalculatorPanel`,
   `window.CalculatorHistory`, `window.CalculatorConverter` and `window.CalculatorDemo`.
2. **`'use strict';` is the first statement of every IIFE.** Nothing precedes it inside the function body.
3. **The IIFE receives `window` as a named parameter** (`global`), so the global object is explicit and the
   module can be read without guessing where `window` comes from.
4. **Dependencies are resolved at the top of the IIFE**, never in the middle of a function. If a dependency
   is missing, the failure happens at load time and is easy to find.
5. **No implicit globals.** Every variable is declared. Assigning to an undeclared identifier is forbidden.
6. **The global object is the only public surface.** Only the members listed in the published object can be
   used from another module; everything else stays private to the closure.
7. **Function declarations are hoisted inside the IIFE and are always declared before the publishing
   object at the bottom.** The module's public API is therefore the last block of the file.
8. **Cross-module calls go through `window`**, never by reaching into another module's internals:
   `window.CalculatorPanel.setMode(...)`, `window.onCalculationStored(...)`.
9. **Load order is documented in `index.html`** and must not be changed casually:

   ```html
   <script src="js/config.js"></script>      <!-- data and settings, no dependencies -->
   <script src="js/api.js"></script>         <!-- needs CALCULATOR_CONFIG -->
   <script src="js/ui.js"></script>          <!-- DOM helpers, needs nothing else -->
   <script src="js/calculator.js"></script>  <!-- needs CalculatorUi, CalculatorApi -->
   <script src="js/history.js"></script>     <!-- needs CalculatorUi, CalculatorApi, CALCULATOR_CONFIG -->
   <script src="js/converter.js"></script>   <!-- needs CalculatorUi, CalculatorApi -->
   <script src="js/demo.js"></script>        <!-- needs CalculatorUi, CalculatorPanel -->
   <script src="js/app.js"></script>         <!-- starts everything, must be last -->
   ```

   Every script is loaded at the end of `<body>` and is **not** marked `defer` or `async`, because the
   order above is the dependency order. `app.js` still waits for `DOMContentLoaded` when the document is
   not parsed yet, so the modules also work if the order of the tags is ever changed.
10. **No module starts itself on load.** `app.js` calls `init()` on the modules it owns, which keeps the
    start-up sequence in one readable place.

## 4. Naming

| Kind | Convention | Example |
| --- | --- | --- |
| Variables and functions | `lowerCamelCase` | `insertOperator`, `openParens`, `renderBaseResults` |
| Module constants | `UPPER_SNAKE_CASE` | `BASIC_KEYS`, `HEALTH_INTERVAL`, `SCRIPTED_TIME`, `DEMO_HISTORY` |
| Published module object | `UpperCamelCase` with the `Calculator` prefix | `window.CalculatorHistory` |
| Cached DOM references | the object is named **`nodes`** | `var nodes = { list: ..., next: ... }` |
| Mutable module state | the object is named **`state`** | `var state = { offset: 0, loading: false }` |
| Boolean values | read as a predicate | `busy`, `loading`, `active`, `dark`, `isActive()`, `backendOnline` |
| Functions that return a boolean | prefixed `is`, `has` or `can` | `isActive()`, `has(...)` inside `find` callbacks |
| Event handlers | prefixed `on` plus the event | `onClick`, `onKeyDown`, `onCategoryChange`, `onVisibility` |
| CSS classes | BEM-inspired, see section 8 | `.history-item__expression`, `.key--operator` |
| DOM ids | `lowerCamelCase` | `expressionInput`, `historyPageInfo`, `unitSwap` |
| `data-*` keys | `lowerCamelCase`, read through `dataset` | `data-tab` → `dataset.tab`, `data-action` → `dataset.action` |
| API payload fields | the names the back end uses, unchanged | `expression`, `resultText`, `historyId`, `fromBase` |

**Abbreviations.** Only four are allowed: `id`, `url`, `api` and `dom`. Anything else is written out —
`expression` rather than `expr`, `message` rather than `msg`, `configuration` rather than `cfg`,
`request`/`response` rather than `req`/`res`. Where a third-party name already contains an abbreviation
(for example the `"api"` query parameter or the `API` in `CALCULATOR_API_BASE`), the external name is kept
as it is.

## 5. Formatting

- **Indentation:** two spaces, never a tab. Continuation lines are indented by two further spaces.
- **Semicolons:** required at the end of every statement. No reliance on automatic semicolon insertion.
- **Quotes:** single quotes for strings. Double quotes only inside HTML attributes and inside a string that
  already contains a single quote (`'Back end unreachable · calculating is disabled'` uses none).
- **One statement per line**, one declaration per statement.
- **Line length:** 110 columns maximum, including comments and JSDoc. Long DOM lookups and long request
  calls are wrapped with the arguments on continuation lines rather than compressed.
- **Braces:** required for every block, opening brace on the same line, closing brace on its own line:
  `if (response.ok) { ... }`, never a single-statement `if` without braces.
- **Parentheses and spacing:** one space around binary operators, no space before the argument list of a
  call, a space after every comma. `if (`, `for (`, `while (` — a space between the keyword and the
  parenthesis.
- **Blank lines between logical blocks:** one blank line between functions, between the sections of a
  function and between the members of a long object literal. No more than one consecutive blank line.
- **Trailing whitespace is not allowed**; every file ends with a single newline.
- **Object literals:** one property per line when there is more than one property, trailing commas are not
  used in function calls and object literals, keys are written without quotes unless they are not valid
  identifiers or contain a hyphen (`'invalid-expression'` in the demo state table).
- **Numeric and string literals:** numbers that carry a meaning are given a named constant
  (`HEALTH_INTERVAL`, `requestTimeout`, `historyPageSize`) instead of being repeated in the code.

## 6. Comments and documentation

- **A module header comment is mandatory.** It is the first thing in the file, it names the responsibility
  of the module in one or two sentences and — where useful — states explicitly what the module does *not*
  do (`api.js` states that no calculation happens there; `calculator.js` states that it never evaluates
  anything).
- **JSDoc for every exported function**, with `@param` for every parameter (including a `[optional]` form)
  and `@returns` when something is returned:

  ```javascript
  /**
   * Loads one page of history from the back end.
   *
   * @param {object} [options]
   * @param {number} [options.offset] overrides the current offset
   * @returns {Promise<boolean>} whether the request succeeded
   */
  async function load(options) { /* ... */ }
  ```

- **Inner functions** get a one-line `/** ... */` comment describing what they return or do; two-line
  wrappers and obvious callbacks may stay uncommented.
- **Module objects and constants are documented** individually (`/** History page size; the back-end
  default is 10 and the maximum 100. */`).
- **English only.** All comments, JSDoc, class names, commit messages and user-visible strings are in
  English, including the text of toasts and error messages.
- **Comments explain *why*, not *what*.** A comment that repeats the next line is deleted. Examples of the
  rule in the existing code: the comment above the function-name-aware backspace explains the user
  expectation, the comment inside `insertOperator` explains why a trailing operator is replaced instead of
  appended, the comment in `history.js` explains why a delete is followed by a re-query.
- **Inline comments are used only where the reason is not visible from the code**, they sit on their own
  line above the code they describe, they start with `// ` and a capital letter.
- **Comment markers in site comments:** the section markers of the stylesheet and of `index.html` use the
  `/* --- */` and `<!-- --- -->` banner form, so the structure of a long file is visible while scrolling.
- **No commented-out code** is committed. Dead code is deleted; version control remembers it.
- **No placeholders** such as `TODO`, `FIXME` or `your-name` are left in a committed file.

## 7. DOM rules

- **Never build markup from data with `innerHTML`.** Dynamic values — expressions, results, unit names,
  translated messages — are written with `textContent` on an element created by `document.createElement`.
  The helper `ui.create(tag, properties, children)` exists for exactly that reason, and `properties.text`
  is applied as `textContent`, never as HTML. A stored expression such as `<img src=x onerror=...>` is
  therefore displayed literally and cannot become markup.
- **Cache element lookups.** A module resolves the elements it needs once into its `nodes` object at load
  time; `document.getElementById` is not called inside render loops or event handlers.
- **Build the keypads from the data tables.** `BASIC_KEYS` (20 keys) and `SCIENTIFIC_KEYS` (30 keys) in
  `ui.js` describe the layout and the behaviour of the same button; `ui.renderKeypad` turns them into DOM
  nodes. A button is added by adding one entry to the table, and the `index.html` containers stay empty.
- **Use `ui.replaceChildren`** to re-render a container instead of appending to it a second time, so
  repeated renders cannot accumulate stale nodes.
- **Listeners are attached with `addEventListener`**, never with `on*` properties or inline attributes.
- **Toggle appearance with `classList`** (`classList.toggle('is-active', ...)`) and small state flags with
  `dataset` (`node.dataset.state = 'offline'`), not by rewriting `className` strings or by setting inline
  styles. Inline styles are reserved for the scripted outlines that the demonstration mode draws.
- **The display value and the API value are different representations of the same expression.** `*`, `/`,
  `-`, `**` are converted for the user with `ui.toDisplayExpression` and converted back with
  `ui.toApiExpression` before any request. No other module is allowed to do this conversion by hand.
- **Do not read or write `localStorage` outside the theme code.** The single key in use is
  `calculator.theme`, handled by `app.js`.
- **Accessibility attributes are part of the markup**, not an afterthought: `aria-label` on every
  icon-only button, `aria-live="polite"` on the `<output>` that shows the result, `role="status"` on the
  toast container, `aria-hidden="true"` on decorative glyphs.

## 8. CSS rules

- **BEM-inspired class names**, in lower-case with hyphens between words:
  - block: `.history-item`, `.stats-cell`, `.converter-card`, `.key`
  - element: `.history-item__expression`, `.history-item__meta`, `.stats-cell__label`, `.brand-mark`
  - modifier: `.key--operator`, `.key--equals`, `.tag--scientific`
  Generic state classes are `is-`-prefixed and are applied by JavaScript: `.is-active`, `.is-visible`.
  Ids are used as JavaScript hooks and are not used as style selectors.
- **Design tokens are custom properties declared once in `:root`** (`--color-surface`, `--color-text`,
  `--space-3`, `--radius-md`, `--shadow-card`, …). Components read the tokens only; a raw colour, radius
  or spacing value inside a component rule is not allowed when a token exists.
- **The dark theme re-declares the same tokens** in a `[data-theme="dark"]` block. It does not restate
  component rules, so a component never needs a theme-specific variant and the two themes cannot drift
  apart.
- **Stylesheet order is fixed and numbered**, and any new section is inserted in the right place:
  1. design tokens, 2. reset and page frame, 3. header, 4. buttons, 5. workspace layout, 6. display,
  7. tabs, 8. keypad, 9. conversion tab, 10. history panel, 11. statistics, 12. demonstration banner,
  13. toast notification, 14. responsive behaviour and reduced motion. Each section starts with the
  banner comment used by the file.
- **`!important` is forbidden**, with one documented exception: the `transition: none !important` inside
  the `@media (prefers-reduced-motion: reduce)` block, where the declaration has to win over the
  component transitions. A screenshot-only stylesheet may use it as well, because it must not be
  overridden by the application stylesheet. Neither exception applies to layout or colour rules.
- **Responsive breakpoints are fixed at 1040 px and 560 px.** At 1040 px the workspace collapses to a
  single column and the history panel loses its fixed height; at 560 px the header stacks, the padding is
  reduced, the keys grow to a 44 px touch target and the result font is reduced. No other breakpoint is
  introduced without updating this document.
- **`@media (prefers-reduced-motion: reduce)` must be respected**: every transition of the interface is
  disabled in that block.
- **No framework and no preprocessor.** No Sass, no Less, no utility-class framework, no reset library
  copied from the internet. Selectors are kept shallow (at most two levels) and no rule targets an
  element by tag alone when a class exists.
- **Long property lists are ordered consistently** — positioning, box model, typography, colour,
  decoration, animation — so that a diff is readable.
- **Vendor prefixes are not written by hand.** The features in use are supported unprefixed by the
  browsers listed in the README.

## 9. API rules

1. **Every request goes through `js/api.js`.** It is the only file that calls `fetch`. A `fetch` call in
   another module is a defect, even if it would work.
2. **Always handle the failure branch.** Every call returns
   `{ ok, data, code, message, status }`; code that reads `response.data` without checking `response.ok`
   first is not accepted.
3. **Show the back-end message to the user.** The `error` code and the `message` of the back end are what
   the message line and the toasts display, for example `EVALUATOR_DIVISION_BY_ZERO · Division by zero`.
   The front end does not invent a replacement wording for a failure that the server described.
4. **Never compute a result in the browser.** No `eval`, no `Function` constructor, no arithmetic on the
   expression string, no local expression parser. The expression is sent as text and the answer is
   rendered as it arrives.
5. **Never cache a calculation result as if it were the source of truth.** After a successful calculation
   the history is re-read from the back end; after a delete the list is queried again. `localStorage` is
   not used for calculation data.
6. **Filtering, pagination and statistics are requested from the back end**, with the documented query
   parameters (`limit`, `offset`, `keyword`, `operation`, `days`), rather than applied to an array held in
   the browser.
7. **A request always has a timeout.** New endpoints are added as methods on `CalculatorApi` so that they
   inherit the `AbortController` timeout and the normalisation of the answer.
8. **The API base URL comes from `CALCULATOR_CONFIG.apiBase`**, through `api.url(path)`; no module builds
   an endpoint address from a hard-coded host.

## 10. Error handling rules

- **Report the back-end code and message; never surface a raw exception.** An unexpected JavaScript error
  is caught at the boundary where the request is made and turned into a short, human-readable message;
  a stack trace or an internal name is never shown in the display, the history list or a toast.
- **A failed request must not break the page.** A failed history load renders a placeholder row, a failed
  statistics load renders a single "Statistics unavailable" cell, and a failed conversion renders its
  error inside the conversion card. The rest of the interface keeps working.
- **Keep the interface usable when the back end is unreachable.** The keypad, the tabs, the theme switch
  and the search field stay operable; the header indicator turns red and the message line explains that no
  new result can be produced. There is no local fallback calculation, by design.
- **Distinguish a rejected request from an unreachable server.** A rejected expression carries its own
  code from the back end; an unreachable server is always reported as `NETWORK_UNREACHABLE` (including the
  timeout case) and is the only code that switches the connection indicator.
- **Guard against double submission.** A request in flight sets the module's `busy` or `loading` flag and
  disables the triggering control, so two identical records cannot be created by an impatient double
  click.
- **Reset the disabled state on every exit path**, including the failure path, so that a control can never
  stay permanently disabled after an error.
- **Empty input is not an error.** Pressing `=` on an empty expression returns silently; a missing value in
  the converter restores the placeholder hint instead of sending a request.

## 11. Accessibility rules

- **`aria-label` on every icon-only control** (`⟳`, `⇄`, `±`, `⌫`, `☾`): the visible glyph is not a name.
  The keypad generated from the data tables sets `aria-label` on each button, adding the API value in
  parentheses when the label is a symbol, for example `× (*)`.
- **The result is an `<output aria-live="polite">`**, so a screen reader announces a new result without
  interrupting the user. The toast container is a `role="status"` live region as well; the message line
  under the result is a regular paragraph and is not announced twice.
- **Focus must stay visible.** The stylesheet draws a focus ring with `color-mix()`; removing an outline
  without providing a replacement is not allowed, and the design keeps the ring visible in both themes.
- **Everything is operable by keyboard.** The keypad buttons are real `<button type="button">` elements in
  tab order, the tabs are buttons, the physical keyboard shortcuts mirror the keypad, and no action
  depends on hover or on a pointer gesture.
- **Semantic elements only.** `header` for the identity and status bar, `main` for the workspace,
  `section` for the calculator and the statistics, `nav` for the tabs, `aside` for the history panel,
  `footer` for the explanatory note, `ul`/`li` for the history list and the keypad rows, `label` for every
  input. A `div` is used only where no semantic element fits.
- **Labels are explicit.** Every control has a visible `<span>` label or an `aria-label`, never a
  placeholder alone.
- **Colour is never the only signal.** The connection state is a colour *and* a text; an error is a red
  message *and* a code; a selected tab is highlighted *and* marked with `is-active`.
- **Text keeps readable contrast in both themes**, because both themes use the same token names with
  tested values.
- **Motion is optional:** the interface is fully usable with `prefers-reduced-motion: reduce`.

## 12. Git commit message convention

Commits follow the widely used Conventional Commits form, with a scope that names the part of the front
end that changed:

```text
<type>(<scope>): <imperative summary in the present tense>

<optional body: why the change is needed, in English, wrapped at 72 columns>

<optional footer: Refs #12>
```

- **Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`.
- **Scopes:** `calc` (calculator panel), `history`, `convert`, `api`, `ui`, `theme`, `demo`, `docs`,
  `build` (deployment or copy into the back-end `public/` folder).
- **Summary:** lower-case, no trailing full stop, at most 72 characters, written in the imperative
  ("add", not "added" or "adds").
- **Body:** one blank line after the summary; explains the reason and the behaviour change, not the
  content of the diff.
- **One logical change per commit.** A change to the front end and the matching change in the back-end
  repository are two commits in two repositories, and the front-end commit says which back-end change it
  depends on.
- **Never commit a generated or copied artefact** (`public/` of the back-end repository, screenshots of a
  local run) with the source change. `config.local.js` is never committed.

Example:

```text
feat(history): re-query the list after deleting one record

The row was previously removed from the DOM only, so a refresh brought the
record back when the delete request had failed. The list is now read again
from the back end after every delete, because the database is the only
authoritative source for the history.

Refs #12
```

## 13. What is not allowed

- **No framework** — no React, Vue, Angular, jQuery or any equivalent. The DOM is used directly.
- **No bundler, transpiler or build step** — no webpack, Rollup, Vite, Babel or TypeScript. `src/` is
  deployed as it is committed.
- **No npm dependency in the front end** — there is no `package.json` in this repository, and no library
  is loaded from a CDN.
- **No `eval`, no `Function` constructor, no `setTimeout` with a string.** Nothing executes data.
- **No `innerHTML` (or `insertAdjacentHTML`, or `outerHTML`) with dynamic data.** Only `textContent` and
  created nodes.
- **No calculation in the front end** — no local parser, no arithmetic on the expression, no fallback
  result when the back end is down.
- **No `fetch` outside `js/api.js`.**
- **No CSS preprocessor, no utility-class framework, no copied reset library, no `!important`** outside
  the two documented exceptions in section 8.
- **No new dependency, polyfill or CDN script without a documented reason** in the pull request
  description and in this document, together with the effect on the "no build step" rule.
- **No `var` in new or edited code**, no implicit globals, no reliance on automatic semicolon insertion.
- **No non-English identifiers, comments, messages or commit messages.**
- **No commented-out code, no `TODO`/`FIXME` marker and no placeholder text** in a committed file.
- **No personal data in the repository** — the student is identified by the student ID `832402111` only.

## 14. How these rules are verified in this project

The rules are checked in three ways, all of them manual by design, because the repository has no
dependency and therefore no tool configuration of its own:

1. **Manual review against this document.** A reviewer reads the diff and checks the sections above:
   module shape and `'use strict'`, naming, the 110-column limit, JSDoc on exported functions, `textContent`
   instead of `innerHTML`, `fetch` only in `api.js`, and the failure branch of every new call.
2. **Syntax check of every JavaScript file with `node --check`** (a syntax check only — the front end does
   not run on Node.js):

   ```bash
   for file in src/js/*.js; do node --check "$file"; done
   ```

   On Windows (PowerShell):

   ```powershell
   Get-ChildItem .\src\js\*.js | ForEach-Object { node --check $_.FullName }
   ```

3. **A browser check of every screen, in both themes and at both breakpoints.** The reviewer opens the
   page, walks through the screens — basic keypad, scientific keypad, both conversion cards, the history
   list with search, filter and pagination, the statistics cells, the toast, the demonstration banner —
   repeats the walkthrough in the light and the dark theme, and narrows the window below 1040 px and below
   560 px to confirm the responsive rules. For screenshots, the scripted states of `?demo=` are used, so
   each image is reproducible.

**Honest limitation.** There is **no ESLint configuration and no Stylelint configuration committed in
this repository yet**, so the formatting and naming rules of sections 4, 5 and 8 are enforced by review
only and are not machine-checked. Adding a committed lint setup (a shared configuration for ESLint and
Stylelint, runnable through `npx` without adding a dependency to `src/`) is a listed future improvement;
the same applies to a small browser-side test page that would exercise the calculator, the history and the
converter against a scripted back end.
