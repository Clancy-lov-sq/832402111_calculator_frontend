# Calculator System — Front End (Student ID 832402111)

Front end of the **"First Assignment — Front-End and Back-End Separation Calculator System"** for the
Software Engineering course, student ID **832402111**. This repository contains only the browser side of
the system; the calculator service lives in the separate repository
[`832402111_calculator_backend`](https://github.com/Clancy-lov-sq/832402111_calculator_backend). The page
never evaluates an expression itself: it sends the raw expression to the back end with
`POST /api/calculate` and renders whatever the back end answers. Every result on the screen — and every
row of the history — is produced by the server, which is the core requirement of the assignment.

---

## Table of contents

- [Features overview](#features-overview)
  - [Required functions](#required-functions)
  - [Extended features](#extended-features)
- [Technology stack](#technology-stack)
- [Runtime environment](#runtime-environment)
- [Installation and start-up](#installation-and-start-up)
  - [Option A - separated deployment (two services)](#option-a---separated-deployment-two-services)
  - [Option B - single-service deployment](#option-b---single-service-deployment)
- [Configuration](#configuration)
- [How the front end talks to the back end](#how-the-front-end-talks-to-the-back-end)
- [Demonstration mode](#demonstration-mode)
- [Project structure](#project-structure)
- [Verification checklist](#verification-checklist)
- [Browser support and known limitations](#browser-support-and-known-limitations)
- [Related repository and license](#related-repository-and-license)

---

## Features overview

### Required functions

| Function | What it actually does |
| --- | --- |
| Basic calculation through the back end | The keypad only builds a string. Pressing `=` calls `POST /api/calculate` and prints the `resultText` field of the JSON answer in the display. No arithmetic exists in the browser. |
| Compound expressions | Multi-term expressions such as `1+2*3` or `(1+2)*3` are sent as text; the parsing rules, the operator precedence and the associativity are the back end's responsibility. |
| History read from the database | The history panel calls `GET /api/history` on load, after every successful calculation and on demand. It renders what the database returned — expression, result, operation tag, timestamp and record id — newest first, at most 6 rows per page. |
| Delete one record | The `Delete` button of a row calls `DELETE /api/history/{id}` and then **queries the list again**. The row disappears because the database no longer contains it, not because the DOM was edited. |

Two supporting behaviours belong to the required part as well:

- **Error reporting.** A rejected expression is displayed in the message line under the result together
  with its stable code, for example `PARSER_MISSING_OPERAND · Operator is missing an operand` or
  `EVALUATOR_DIVISION_BY_ZERO · Division by zero`. The code and the wording come from the back end.
- **Offline tolerance.** If the back end is stopped, the interface stays usable: the header indicator
  turns red, the message line explains that no new result can be produced, and the history panel reports
  that it is unavailable. Nothing is calculated locally as a fallback.

### Extended features

| Extended feature | What it actually does |
| --- | --- |
| Scientific mode | The **Scientific** tab renders a second keypad of 30 keys (functions such as `sin(`, `cos(`, `tan(`, `ln(`, `log(`, `sqrt(`, the constants `π` and `e`, `xʸ`, `x²`) and switches the evaluation mode sent to the back end from `basic` to `scientific`, so that function names are accepted. The expression is still evaluated by the server. |
| Number base conversion | The **Conversion** tab converts one value between decimal, binary, octal, hexadecimal and base 36 by calling `POST /api/convert/base`, and lists every base the back end returned. |
| Unit conversion | The same tab converts a value between units — for example length, temperature and speed — by calling `POST /api/convert/unit`. The category and unit lists come from `GET /api/convert/units`, and a **swap** button exchanges the *From* and *To* units and converts again. |
| History search, filter and pagination | The search box sends a debounced (320 ms) `keyword` parameter, the drop-down sends an `operation` parameter, and the **Newer** / **Older** buttons send `offset` with a fixed `limit` of 6. All four are query parameters handled by the back end, not client-side filtering of a cached array. |
| Statistics | The cells under the history list are filled from `GET /api/stats?days=7`: total calculations, a count per operation family and the count for today. A failure here degrades to a single "Statistics unavailable" cell and never blocks the list. |
| Dark theme | The header button toggles `data-theme="light|dark"` on the root element. The dark theme re-declares the same design tokens, so no component needs a second rule set. On the first visit the stored choice is used, otherwise `prefers-color-scheme` decides. |
| Keyboard shortcuts | The physical keyboard drives the same actions as the on-screen keypad: digits and `.`, the operators `+ - * / % ^ ( )`, `Enter` to evaluate, `Escape` to clear and `Backspace` to delete. The expression field can also be typed into directly, and `×` / `÷` typed by the user are converted to `*` / `/` before the request is sent. |
| Connection indicator | The header polls `GET /api/health` every 15 s and shows a coloured dot with text such as *Back end online · &lt;database&gt; · &lt;n&gt; records*, or *Back end unreachable · calculating is disabled*. A network failure reported by any request switches the indicator immediately, without waiting for the next poll. |

---

## Technology stack

| Layer | Technology | Notes |
| --- | --- | --- |
| Markup | HTML5 | `src/index.html` is the only page: `header`, `main`, `section`, `nav`, `aside`, `footer`, `<output>` for the result. |
| Styling | CSS3 | `src/css/style.css`: custom properties as design tokens, CSS grid and flexbox layout, `color-mix()` for a few tints, no framework. |
| Scripting | ES2015+ JavaScript | Classic `<script>` tags, one IIFE per module, `async`/`await`, `const`/`let`, template-free string building. |
| HTTP | Fetch API | Every call to the back end goes through `window.fetch` inside `src/js/api.js`. |
| Timeouts | AbortController | Each request is aborted after `requestTimeout` (8000 ms) so a dead server cannot freeze the interface. |
| Storage | localStorage | Used for **the theme only**, under the key `calculator.theme`. The history is never cached in the browser. |
| Build tooling | *None* | No framework, no npm dependency, no bundler, no transpiler, no build step. |

**Why there is no build step.** The `src/` folder is deployed exactly as it is committed: the same files
are read during a code review, opened by a static server and copied into the back-end `public/` folder.
A reviewer can read the whole front end without installing anything, and the deployment stays a plain
file copy instead of a toolchain that has to be reproduced.

---

## Runtime environment

- **Any modern browser** — see [Browser support](#browser-support-and-known-limitations). The page is
  served over HTTP; only `fetch`, `AbortController`, CSS custom properties, `color-mix()` and ES2015+
  syntax are required.
- **No Node.js and no npm are required for the front end itself.** Node.js is needed only for the back
  end, which is a separate repository.
- **A static file server for the separated deployment**, or nothing at all for the single-service
  deployment, where the back end serves the page and the API on the same origin.
- The back end is expected on `http://127.0.0.1:8080` during local development.

---

## Installation and start-up

### Option A - separated deployment (two services)

This is the normal development setup: the page is served by a static server and the API by the back-end
service, on two different ports of the same machine.

1. **Start the back end first** (in the back-end repository, which listens on port 8080):

   ```bash
   git clone https://github.com/Clancy-lov-sq/832402111_calculator_backend.git
   cd 832402111_calculator_backend
   npm install
   npm start
   ```

   The service answers on `http://127.0.0.1:8080`; `GET http://127.0.0.1:8080/api/health` should return
   a JSON object with `"success": true`.

2. **Serve this front end.** From the repository root (the folder that contains `src`):

   ```bash
   python -m http.server 5500
   ```

   or, serving the `src` folder directly:

   ```bash
   npx serve src
   ```

   An equivalent alternative is the VS Code **Live Server** extension: right-click `src/index.html` →
   *Open with Live Server*.

3. **Open the page.** With the server started in the repository root (`python -m http.server 5500`) the
   address is `http://127.0.0.1:5500/src/index.html`. With a server started inside `src/`
   (`npx serve src`, or Live Server on `src/index.html`) the page is at the root of that server instead —
   `http://127.0.0.1:3000/index.html` for `npx serve` on its default port, or whatever address the tool
   prints at start-up. The port does not matter to the front end, because the API address is configured
   separately. The header indicator turns green within a moment; if it stays red, see
   [Configuration](#configuration).

### Option B - single-service deployment

The back end can serve the page and the API from one origin, which removes every cross-origin question:

1. Copy the content of `src/` into the `public/` folder of the back-end repository:

   ```bash
   # from the repository root, with both repositories side by side
   cp -r src/* ../832402111_calculator_backend/public/
   ```

   On Windows (PowerShell):

   ```powershell
   Copy-Item -Recurse -Force .\src\* ..\832402111_calculator_backend\public\
   ```

2. Start the back end (`npm start` on port 8080) and open `http://127.0.0.1:8080`.

   The page is now on the same origin as the API. `js/config.js` detects that it is served by the back
   end and uses relative URLs, so nothing has to be configured and no CORS request is made. Re-copy
   `src/` after every front-end change, because `public/` is a copy and not a link.

---

## Configuration

The address of the API is resolved by `src/js/config.js` when the page loads, in this order:

| Priority | Source | When it is used |
| --- | --- | --- |
| 1 | The `?api=` URL parameter | An explicit override, for example to point the same build at a local or a deployed back end. A trailing slash is removed. |
| 2 | `window.CALCULATOR_API_BASE` | Injected by an optional `config.local.js` that is **not** committed, so a deployment can be adapted without touching source code. |
| 3 | Automatic detection | If the page is not served from `127.0.0.1` / `localhost`, it is assumed to be served by the back end and the API base becomes empty (relative URLs on the same origin). On a local host with a port other than `8080`, the local back end at `http://127.0.0.1:8080` is used. |

The resulting value is published as `CALCULATOR_CONFIG.apiBase`, together with `requestTimeout` (8000 ms),
`historyPageSize` (6), the storage keys (`calculator.theme`, `calculator.mode`) and a version string.

**Override example** — the static server on port 5500, the back end somewhere else:

```text
http://127.0.0.1:5500/src/index.html?api=http://127.0.0.1:8081
```

**Adapting a deployment without editing source code.** Create a file `src/js/config.local.js` that sets
the address, and load it in `index.html` *before* `js/config.js`:

```html
<script src="js/config.local.js"></script>
<script src="js/config.js"></script>
```

```javascript
// js/config.local.js — not committed; only this file differs per deployment
window.CALCULATOR_API_BASE = 'https://calculator-api.example.edu';
```

**Do not open `index.html` directly from the file system.** With a `file://` URL the origin is opaque,
`localStorage` and `fetch` behave differently in every browser, and requests to `http://127.0.0.1:8080`
are blocked as cross-origin. Always serve the folder over HTTP, as described above.

---

## How the front end talks to the back end

`src/js/api.js` is the only module that calls `fetch`. It normalises every answer — success or failure —
into one object shape, so no other module has to know about HTTP status codes or JSON parsing:

```javascript
{ ok: true,  data: {...}, code: null,  message: null,  status: 200 }  // success
{ ok: false, data: {...}, code: '...', message: '...', status: 4xx }  // rejected
{ ok: false, data: {},    code: 'NETWORK_UNREACHABLE', message: 'Cannot reach the back end (…)', status: 0 }
```

### Endpoints used by the interface

| Endpoint | Method | Used for |
| --- | --- | --- |
| `/api/calculate` | POST | Evaluate the expression typed in the display; the back end also stores the calculation. |
| `/api/history` | GET | Read one page of history; accepts `limit`, `offset`, `keyword` and `operation`. |
| `/api/history/{id}` | DELETE | Delete exactly one database record. |
| `/api/history` | DELETE | Delete every record (*Clear all* button). |
| `/api/stats?days=7` | GET | Statistics cells under the history list. |
| `/api/health` | GET | Connection indicator, polled every 15 s. |
| `/api/capabilities` | GET | Available operators, functions, constants and bases (exposed by `api.js`). |
| `/api/convert/base` | POST | Number base conversion. |
| `/api/convert/unit` | POST | Unit conversion. |
| `/api/convert/units` | GET | The category and unit catalog that fills the conversion selects. |

All paths are prefixed with the resolved API base, for example
`http://127.0.0.1:8080/api/calculate`.

### Calculation request and response

Request — `POST /api/calculate` with `Content-Type: application/json`:

```json
{
  "expression": "(1+2)*3",
  "mode": "basic",
  "ans": 20
}
```

`expression` is the raw text in the API representation (`*`, `/`, `-`, `**`), `mode` is `basic` or
`scientific` and follows the active tab, and `ans` is the previous result, sent only when one exists so
that the `ans` constant can be used.

Success response (`HTTP 200`, `"success": true`):

```json
{
  "success": true,
  "expression": "(1+2)*3",
  "result": 9,
  "resultText": "9",
  "operation": "compound",
  "mode": "basic",
  "durationMs": 0.28,
  "historyId": 41,
  "createdAt": "2026-10-03 09:41:26"
}
```

The display prints `resultText` (falling back to `result`), the message line reports the back-end
duration, and the small badge next to the mode shows `saved as #41` when `historyId` is present. A
successful calculation is also the trigger to reload the history panel, because the database changed.

### Failure response

Every rejected request is answered with the same three fields, and the interface shows the code and the
message to the user — never a raw exception:

```json
{
  "success": false,
  "error": "EVALUATOR_DIVISION_BY_ZERO",
  "message": "Division by zero"
}
```

It is rendered as `EVALUATOR_DIVISION_BY_ZERO · Division by zero` in the message line under the result,
while the result area shows `Error`. If the server cannot be reached at all — stopped, wrong port or
timeout after 8000 ms — the front end synthesises the code `NETWORK_UNREACHABLE` with the API base in the
message; that code is what turns the header indicator red.

---

## Demonstration mode

The URL parameter `?demo=<state>` puts the interface into a fixed, scripted state. It exists so that the
**screenshots in the submission material are reproducible**: the same URL always produces exactly the
same display, the same history rows and the same statistics, instead of depending on which records happen
to be in the database or on how fast the machine is. The images shipped with the assignment are
1600 × 1000 PNG files taken from the real running project in these states.

While a state is active, a banner at the top of the page explains that the values shown are part of a
scripted demonstration. `js/api.js` asks `CalculatorDemo.getScriptedResponse(path, method)` before sending
a request; the module returns the scripted answer for that state, or `null` to let the request through.

Available states:

| State | Shows |
| --- | --- |
| `addition` | `12+8` = 20 |
| `subtraction` | `20-7` = 13 |
| `multiplication` | `6*7` = 42 |
| `division` | `84/12` = 7 |
| `decimal` | `12.5/2` = 6.25 |
| `precedence` | `1+2*3` = 7 |
| `parentheses` | `(1+2)*3` = 9 |
| `unary` | `3*-2` = −6 |
| `history` | The history panel with the scripted records, first row highlighted |
| `invalid-expression` | The rejected expression `1+×2` with `PARSER_MISSING_OPERAND` |
| `division-by-zero` | The rejected expression `10÷(3−3)` with `EVALUATOR_DIVISION_BY_ZERO` |
| `scientific` | The scientific tab with `sqrt(144)+2^10/4` = 268 |
| `base-conversion` | The conversion tab with 255 in five bases |
| `unit-conversion` | The conversion tab with 100 °C = 212 °F |
| `statistics` | The statistics cells of the scripted history, panel highlighted |
| `history-cleared` | The history panel right after *Clear all* |
| `offline` | A red indicator and the `NETWORK_UNREACHABLE` message of a stopped back end |
| `dark` | The dark theme with `(2+3)*(7−4)` = 15 |

Example URL (static server on port 5500):

```text
http://127.0.0.1:5500/src/index.html?demo=scientific
```

In the scripted `offline` state the connection indicator is pinned red and the periodic health check is
stopped, so the state cannot be overwritten by the real service a moment later.

**Demonstration mode never affects normal operation.** With no `?demo=` parameter in the URL — and for
any value that is not one of the state names above — `CalculatorDemo.isActive()` is `false`, the scripted
branch in `api.js` is skipped and every request goes to the real back end. The layout, the styles, the
button behaviour and every other code path are the same ones used in normal operation in all cases; only
the content of the display, the history rows, the statistics cells and the health answer are scripted.

---

## Project structure

```text
832402111_calculator_frontend/
├── README.md                  this document
├── codestyle.md               the coding standard applied to this repository
└── src/                       the deployable front end (copy this folder as it is)
    ├── index.html             page shell: header, display, three tabs, history panel, toast container
    ├── css/
    │   └── style.css          design tokens, dark theme and every component style, in 14 sections
    └── js/
        ├── config.js          resolves the API base URL and publishes the shared settings
        ├── api.js             the only module that talks to the back end (fetch, timeout, normalisation)
        ├── ui.js              DOM helpers, expression formatting, toast, connection indicator, keypad tables
        ├── calculator.js      expression state, keypad actions, keyboard shortcuts, submit()
        ├── history.js         history list, delete, clear all, search, filter, pagination, statistics
        ├── converter.js       number base and unit conversion tabs, unit catalog, selectUnits()
        ├── demo.js            optional scripted demonstration mode for reproducible screenshots
        └── app.js             bootstrap: modules, tabs, theme, health polling, global callbacks
```

The scripts are loaded in dependency order at the end of `index.html`:
`config.js`, `api.js`, `ui.js`, `calculator.js`, `history.js`, `converter.js`, `demo.js`, `app.js`.
Each file publishes exactly one global object (`CALCULATOR_CONFIG`, `CalculatorApi`, `CalculatorUi`,
`CalculatorPanel`, `CalculatorHistory`, `CalculatorConverter`, `CalculatorDemo`) and reads its
dependencies from `window` at the top of its IIFE.

---

## Verification checklist

A reviewer can follow these steps in order; steps 1–7 need only the calculator, steps 8–10 need the
history, and step 11 needs the back end to be stopped.

| # | Step | Expected result |
| --- | --- | --- |
| 1 | Basic operations. Press `7`, `+`, `8`, `=`. | `15` appears in the result area and the message line reports the back-end duration. Repeat for `−`, `×`, `÷`. |
| 2 | Decimals. Press `1`, `2`, `.`, `5`, `÷`, `2`, `=`. | `6.25`. A second `.` in the same number is not accepted as part of a new number. |
| 3 | Precedence. Type `1+2*3` and press Enter. | `7`, not `9`: the precedence is applied by the back end. |
| 4 | Parentheses. Type `(1+2)*3` and press Enter. | `9`. The `( )` key inserts an opening or a closing parenthesis according to what is still open. |
| 5 | Unary minus. Type `3*-2` and press Enter. | `−6`. The `±` key wraps the last number in parentheses, e.g. `3*(-2)`. |
| 6 | Invalid expression. Type `1+*2` and press Enter. | The result area shows `Error` and the message line `PARSER_MISSING_OPERAND · …` with the back-end wording. |
| 7 | Division by zero. Type `10/(3-3)` and press Enter. | `EVALUATOR_DIVISION_BY_ZERO · Division by zero` — a rejected calculation, not a crash and not `Infinity`. |
| 8 | History after a refresh. Evaluate two or three expressions, then reload the page (F5). | All of them are still listed, newest first, with expression, result, operation tag, timestamp and `#id`. |
| 9 | Delete a record. Press `Delete` on one row, then reload the page. | A toast confirms the deletion, the list is re-read from the back end, and the record is still gone after the reload. |
| 10 | Search, filter and pagination. Type a keyword in the search box; choose an operation in the drop-down; press `Older` / `Newer`. | The list is filtered by the back end, the page footer shows `Showing a–b of n`, and `Newer` is disabled on the first page. |
| 11 | Stop the back end and try to calculate. Stop the service and evaluate any expression. | The header indicator turns red with *Back end unreachable · calculating is disabled*; the message line shows `NETWORK_UNREACHABLE` with the API address; the history reports that it is unavailable; the keypad, the tabs and the theme switch keep working but no new result appears. Restart the service: the next health poll turns the indicator green again. |
| 12 | Extended features. Open the **Scientific** tab (`sqrt(144)+2^10/4`), the **Conversion** tab (base 255 → bin/oct/hex, 100 °C → °F, swap the units), toggle the **dark theme**, and use the keyboard (`Enter`, `Escape`, `Backspace`). | Each value appears as the answer of the corresponding back-end endpoint; the theme survives a reload; the keyboard behaves like the on-screen keypad. |
| 13 | Screenshot states (optional). Open `index.html?demo=addition`, then any other state from the list above. | The scripted state appears with the demonstration banner on top, and a reload of the plain `index.html` shows no trace of it. |

---

## Browser support and known limitations

**Supported browsers.** The page targets current versions of Chrome, Edge, Firefox and Safari. It relies
on `fetch`, `AbortController`, `URLSearchParams`, CSS custom properties, CSS grid and flexbox,
`async`/`await` and optional chaining-free ES2015+ syntax. Internet Explorer is not supported.

**Known limitations, stated honestly:**

- `color-mix()` is used for a few tints (focus rings, a few borders and one shadow). It requires a
  browser from **2023 or newer**; in an older browser those tints fall back to the plain token colour and
  the interface stays usable, but the focus outline is less visible.
- **The arithmetic is intentionally absent from the front end.** The browser builds a string and displays
  an answer; it cannot produce a result on its own, and there is no local fallback evaluator. This is the
  requirement of the assignment, not an oversight.
- **The theme is the only thing kept in `localStorage`** (key `calculator.theme`). The history is never
  cached: every list, page, search and statistic is a fresh request, so the database stays the single
  source of truth. The configured key `calculator.mode` is reserved and currently unused.
- If the back end is unreachable, the history panel shows a placeholder instead of the last known list,
  and the conversion tab cannot fill its unit catalog.
- The history page size (6) and the request timeout (8000 ms) are constants in `src/js/config.js`; they
  are not exposed in the interface.
- The front end has no automated test suite of its own — it is verified manually and by the back-end
  test suite, because the calculations it displays are produced by the server.

---

## Related repository and license

- **Back end (separate repository):**
  [`832402111_calculator_backend`](https://github.com/Clancy-lov-sq/832402111_calculator_backend) — the
  service that parses and evaluates every expression, stores the history in its database and exposes the
  `/api` endpoints listed above. It is a Node.js project started with `npm start` on port 8080.
- **This repository:** `832402111_calculator_frontend` — the static page described in this document. It
  contains no evaluation logic by design.
- **License:** MIT, matching the back-end repository. The code was written by the student for the
  "First Assignment — Front-End and Back-End Separation Calculator System" of the Software Engineering
  course, student ID 832402111, and may be reused for study purposes with attribution.
