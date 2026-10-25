---
name: development-conventions
description: Build or modify web frontends using Flask, offline Bootstrap 5, and separate static HTML, CSS, and JavaScript files. Apply these default frontend rules when creating web interfaces, forms, calculators, and dashboards for the user. Also apply the terminal output palette when creating or modifying shell scripts, Python scripts, command-line scripts, and CLI programs, including Go programs. Use appropriate, idiomatic design patterns when writing or modifying Python and Go code.
---

# Development Conventions

Follow these defaults. Apply the web interface sections to web projects, the independent CLI colors section to scripts and terminal programs, and the design patterns section to Python and Go code. Deviate only when the user explicitly requests a specific exception.

## Existing Projects and Change Scope

- When modifying an existing project, apply these conventions within the scope of the requested change.
- Preserve existing architecture, frameworks, file organization, and working behavior outside that scope. Do not automatically migrate or restructure the entire codebase to match these conventions.
- Make broader migrations or refactoring only when the user requests them or they are necessary to complete the requested change. Keep necessary supporting changes limited and explain their purpose.

## Technology and Offline Operation

- Use a Flask backend and Bootstrap 5.3.x frontend; pin an exact version.
- Bundle Bootstrap CSS and `bootstrap.bundle.min.js` locally under `static/vendor/bootstrap/`. The bundle includes Popper.
- Serve every runtime frontend dependency from local files, including icons and fonts. Use system fonts when custom fonts are unnecessary.
- Do not use CDNs, remote fonts, external scripts, analytics, or frontend resources requiring an internet connection. The interface must work without internet access while the local Flask server is running.
- Separate initial dependency acquisition from runtime operation. If local Bootstrap files are missing and cannot be obtained, report the missing dependency; do not substitute CDN references.

## File Structure

Keep Python, HTML, CSS, and JavaScript separate. Apply the requirement to place `index.htm` in the static directory literally:

```text
project/
  app.py
  requirements.txt
  static/
    index.htm
    css/
      app.css
    js/
      theme.js
      app.js
    vendor/
      bootstrap/
        css/bootstrap.min.css
        js/bootstrap.bundle.min.js
```

- Serve `static/index.htm` at `/` using Flask's `send_static_file("index.htm")`.
- Store additional pages as separate `.htm` files in `static/`.
- Do not embed complete HTML pages in Python strings or put application logic in inline HTML event handlers.
- Use `/static/...` references in static HTML; do not leave unprocessed Jinja expressions in it.
- Exchange dynamic data through Flask API endpoints using structured JSON. Keep the computation engine on the server, in a separate Python module when appropriate.
- Do not place secrets, configuration keys, or server-side files in the publicly accessible `static/` directory.

## Light and Dark Modes

- Always show light mode on the first visit: `<html data-bs-theme="light">`.
- Always provide a clearly visible, keyboard-accessible theme toggle in the navigation bar.
- Use Bootstrap's `data-bs-theme="light"` and `data-bs-theme="dark"` support.
- Optionally remember the user's explicit choice using `localStorage`. Without a stored choice, always default to light mode regardless of the system theme.
- Keep the toggle working if storage fails. Apply a previously selected theme before the first render where possible, using a separate local `theme.js` file.
- Check readability, focus indicators, error messages, and all custom components in both themes.

## Standard Navigation Bar

- Use Bootstrap's standard `navbar` component on every page, for example with `navbar navbar-expand-lg bg-body-tertiary` classes.
- Include a brand or application title, `navbar-nav` items, and an active-page indicator (`aria-current="page"`).
- On narrow screens, use standard `navbar-toggler` and `collapse navbar-collapse` behavior with correct `aria-controls`, `aria-expanded`, and an accessible label.
- Keep navigation order, appearance, and the theme toggle consistent across pages.

## Readable, Non-overlapping Form Layout

- Size and position every textbox and dropdown so its content remains readable. Keep labels, values, help text, and validation messages clear of adjacent controls.
- Use Bootstrap grid layouts, responsive columns, and consistent gaps. Stack controls on narrow screens instead of allowing them to overlap.
- Allow sufficient width and height for expected content, including long dropdown options, units, and localized labels. Avoid clipping selected values or hiding meaningful text with ellipses; use a wider control or a stacked layout when needed.
- Allow labels, help text, and error messages to wrap without colliding with controls. Avoid fixed heights that cut off text and absolute positioning for form layout.
- Verify forms with the longest expected values, visible validation errors, mobile widths, and 200% browser zoom. Ensure controls remain readable and never overlap.

## Numeric Values: Always Use Dropdowns

- Use a native `<select class="form-select">` control for every user-entered numeric value.
- Do not use editable numeric text fields, `input type="number"`, or sliders for numeric entry.
- Determine allowed values, units, and increments from the task. Do not invent arbitrary business or physical constraints.
- Use dependent dropdowns for large value sets. If the required precision or continuous range cannot reasonably be handled this way, clarify the requirement with the user before departing from the dropdown rule.
- Provide a clear label, an invalid initial “Select…” option when appropriate, and visible units in the label.
- Validate submitted values against allowed options on the server; a dropdown is not a security boundary.

## Defensive Input Handling and Engine Protection

- Define each field's type, required status, allowed values or range, size limit, and cross-field constraints.
- Provide immediate client-side feedback, but enforce every rule again in Flask before invoking the engine.
- Validate request content type and schema; reject missing required fields, unknown fields, and incorrectly typed fields. Configure an appropriate request size limit.
- Parse numbers strictly and validate finiteness, range, increments, and allowed options. Do not accept `NaN`, infinity, booleans, or malformed values as numbers.
- Protect the engine against division by zero, invalid mathematical domains, overflow, excessive workloads, and incompatible parameters.
- Do not invoke the engine or change state for invalid input. Do not silently replace invalid input with another value.
- Return structured JSON errors with appropriate HTTP status codes; display clear, field-specific messages. Do not expose stack traces or internal data.
- Do not use `eval` to process input. Use parameterized database queries; do not construct shell commands from user text.
- Render user text safely, for example with `textContent`, rather than unchecked `innerHTML`.
- Protect state-changing requests against CSRF when using cookie-based sessions. Do not enable unnecessary CORS access.
- Prevent accidental duplicate submissions while requests are pending; restore interface usability after network or engine failures.

## Web Interface Pre-delivery Checks

1. Verify that all frontend resources load locally and the application works without internet access.
2. Check the default light theme, theme switching, and remembered choices.
3. Test navbar collapse at mobile widths and keyboard operation of forms. Check textboxes and dropdowns with long content, validation messages, and 200% zoom; confirm that content remains readable and controls never overlap.
4. Confirm that all numeric inputs are dropdowns and pages are separate `static/*.htm` files.
5. Send invalid, missing, tampered, and boundary-value inputs directly to the API; verify that invalid input never invokes the engine.
6. Verify successful processing of at least one valid request and recovery after errors.

---

## CLI colors

Apply this palette to shell scripts, Python scripts, command-line scripts, and CLI programs, including Go programs. These terminal rules also apply independently of any web frontend; the Flask and Bootstrap requirements apply only to web interfaces.

| Output role | Hex color | RGB |
| --- | --- | --- |
| Primary headings and emphasized actions | `#337ab7` | `51, 122, 183` |
| Informational messages and progress | `#5bc0de` | `91, 192, 222` |
| Success messages | `#5cb85c` | `92, 184, 92` |
| Warnings | `#f0ad4e` | `240, 173, 78` |
| Errors and failures | `#d9534f` | `217, 83, 79` |

- Use these exact RGB values through ANSI true-color foreground sequences (`ESC[38;2;R;G;Bm`) when supported. Reset formatting after each colored segment with `ESC[0m`.
- Keep ordinary body text in the terminal's default foreground color. Use the palette consistently for the semantic roles above.
- Centralize color definitions in a reusable helper or constants rather than scattering escape codes throughout the program.
- Respect `NO_COLOR`, explicit color settings, and terminal capabilities. In automatic mode, disable colors for redirected or piped output and unsupported terminals; check the actual destination stream.
- If true color is unavailable, use a suitable supported approximation or plain text. Do not claim an approximation matches the exact hex value.
- Keep machine-readable output such as JSON and CSV free of ANSI escape sequences.
- Include readable text labels such as INFO, OK, WARNING, and ERROR so meaning never depends on color alone.

### CLI Output Checks

1. Verify that each message role uses its assigned palette color on true-color terminals.
2. Verify formatting resets after each colored segment.
3. Verify plain-text fallback on unsupported terminals and when `NO_COLOR` is set.
4. Verify that automatic coloring is disabled for redirected or piped output, checking each destination stream independently.
5. Verify that JSON, CSV, and other machine-readable output contain no ANSI escape sequences.
6. Verify that message labels communicate meaning without color.

---

## Design patterns

- Where practical and useful, use established design patterns in Python and Go to improve clarity, maintainability, reuse, and testability.
- Choose patterns that solve an actual problem in the task. Prefer straightforward functions and composition when a pattern would add unnecessary complexity; do not introduce speculative abstractions or deep inheritance hierarchies.
- Keep input validation, business or computation logic, and presentation or output responsibilities separate.
- Consider Strategy for interchangeable algorithms, Factory for meaningful construction choices, Adapter for external integration boundaries, and Repository when a persistence abstraction is warranted.
- In Python, implement patterns idiomatically with functions, callables, modules, composition, and protocols where appropriate. Use a Flask application factory when application configuration or lifecycle warrants it.
- In Go, prefer composition, explicit dependency injection through constructors or function parameters, and small interfaces defined where they are consumed. Avoid translating class-heavy patterns directly from other languages.
- Briefly explain any nontrivial pattern choice in the implementation summary, including the concrete problem it solves. Do not add a pattern solely to satisfy a checklist.

---
