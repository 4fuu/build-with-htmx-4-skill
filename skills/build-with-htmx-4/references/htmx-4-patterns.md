# htmx 4 Implementation Patterns

These patterns target `htmx.org@4.0.0-beta6` only. Because htmx 4 is a prerelease, verify the official reference when the installed version differs or a detail is not covered here.

## Contents

1. [Start from a complete contract](#start-from-a-complete-contract)
2. [Return HTML, not client rendering instructions](#return-html-not-client-rendering-instructions)
3. [Handle forms and errors](#handle-forms-and-errors)
4. [Update multiple regions](#update-multiple-regions)
5. [Use explicit inheritance](#use-explicit-inheritance)
6. [Choose swaps and preserve state](#choose-swaps-and-preserve-state)
7. [Control concurrency and loading](#control-concurrency-and-loading)
8. [Use events and JavaScript narrowly](#use-events-and-javascript-narrowly)
9. [Handle navigation and history](#handle-navigation-and-history)
10. [Load extensions deliberately](#load-extensions-deliberately)
11. [Preserve security and caching](#preserve-security-and-caching)
12. [Debug and test](#debug-and-test)

## Start from a complete contract

A common progressive-enhancement form is:

```html
<form action="/contacts" method="post"
      hx-action="/contacts"
      hx-target="#contact-list"
      hx-swap="beforeend"
      hx-disable="find button">
  <label>Name <input name="name" required></label>
  <button>Create contact</button>
</form>

<ul id="contact-list">...</ul>
```

`hx-action` can use the form's native `method`; `hx-method` can override it. Use `hx-get`, `hx-post`, and other verb attributes when a native fallback is unnecessary and the method is fixed.

Do not add `hx-trigger` for ordinary clicks, form submissions, or input changes unless the default trigger is wrong. Extra attributes should encode a real behavioral decision.

## Return HTML, not client rendering instructions

The handler should return the exact HTML the chosen swap consumes. For `beforeend` on a `<ul>`, return a rendered `<li>`, not JSON, a full page, or an envelope such as `{ html: "..." }`.

When one GET URL serves both modes:

- without `HX-Request: true`, render the full layout;
- with it, render the fragment required by the target;
- add `Vary: HX-Request` when the response can be cached.

`HX-Request-Type` is `partial` for targeted swaps and `full` for body-level or `hx-select` requests. `HX-Source` and `HX-Target` use `tag#id` descriptions. Use these headers for rendering and diagnostics, not authorization.

Prefer shared partial templates:

```text
page template -> includes contact-list partial
htmx GET      -> renders contact-list partial
htmx POST     -> renders contact-row partial
```

## Handle forms and errors

htmx 4 swaps every response status except `204` and `304` by default. An expected `422` therefore needs HTML that is safe for the selected target, or explicit status routing:

```html
<form hx-action="/profile" method="post"
      hx-target="#profile-card"
      hx-swap="outerHTML"
      hx-status:422="target:#profile-errors select:#validation-errors swap:innerHTML">
  ...
</form>

<div id="profile-errors" role="alert"></div>
```

The server may return a replacement profile card on success and a response containing `#validation-errors` with status `422`. Use exact status rules before wildcard rules; `4xx` and `5xx` are supported. Use `swap:none` only when intentionally suppressing response content.

Keep validation and authorization on the server even when native constraint validation improves feedback. Return meaningful statuses and user-safe fragments. A `500` fragment must not expose stack traces or internal data.

For deletes that need values beyond the URL, remember that htmx 4 `hx-delete` does not automatically include the enclosing form. Add `hx-include="closest form"` only when those values are required.

## Update multiple regions

Use one primary swap plus the smallest number of additional updates.

For a simple ID-based update, return an out-of-band element:

```html
<li id="contact-42">Ada</li>
<span id="contact-count" hx-swap-oob="outerHTML">8</span>
```

Use `<hx-partial>` when targeting by selector, choosing different swap modes, or making routing explicit:

```html
<hx-partial hx-target="#flash" hx-swap="innerHTML">
  <p role="status">Saved</p>
</hx-partial>
<hx-partial hx-target="#contact-count">8</hx-partial>
<li id="contact-42">Ada</li>
```

An `id` on `<hx-partial>` is shorthand for targeting that ID. If a response contains only `<hx-partial>` elements, the primary target is left unchanged unless the triggering swap opts into empty swapping.

The main content swaps first. Out-of-band and partial updates then run in response document order. Keep updates independent rather than relying on one swap to create a target needed by an earlier operation.

## Use explicit inheritance

htmx 4 attributes do not implicitly flow to descendants. Mark intentional inheritance:

```html
<section hx-confirm:inherited="Delete this item?">
  <button hx-delete="/items/42">Delete</button>
</section>
```

Use `:append` to add to an inherited value and `:merge` only where the attribute supports merging. Prefer placing an attribute on the requesting element when inheritance does not reduce repetition or express a clear region policy.

Do not use removed `hx-inherit` or `hx-disinherit` in native htmx 4 code.

## Choose swaps and preserve state

Use the smallest stable boundary:

| Need | Typical swap |
| --- | --- |
| Replace target children | `innerHTML` |
| Replace target itself | `outerHTML` |
| Add a row or message | `beforeend` / `afterbegin` |
| Remove target | `delete` |
| Replace text without HTML | `textContent` |
| Preserve stable DOM and input state | `innerMorph` / `outerMorph` |

Prefer ordinary swaps. Use morphing only when focus, selection, input state, media, or third-party DOM identity must survive. Give stable nodes purposeful IDs and test the behavior in a browser.

`hx-preserve` is for an element that must survive a replacement; it is not a substitute for choosing the correct target. Clean up third-party widgets before removed nodes and initialize them after new content is processed.

## Control concurrency and loading

Use `hx-sync` only when requests can race:

```html
<input type="search" name="q"
       hx-get="/search"
       hx-trigger="keyup changed delay:300ms, search"
       hx-target="#results"
       hx-sync="this:replace">
```

Common strategies are `drop`, `abort`, `replace`, and `queue first|last|all`. `replace` fits latest-result-wins search. `closest form:abort` can cancel field validation when the form submits.

Use `hx-disable="this"` or a scoped selector to block duplicate clicks while a request is active. Use `hx-indicator` and the project's existing loading styles for feedback.

These controls do not ensure durable correctness. Use server-side transactions, constraints, idempotency keys, or version checks when duplicate or concurrent requests can corrupt data.

## Use events and JavaScript narrowly

htmx 4 uses `fetch()`, not `XMLHttpRequest`. Event names use `htmx:<phase>:<action>`, for example:

- `htmx:config:request`
- `htmx:before:request`
- `htmx:after:request`
- `htmx:before:swap`
- `htmx:after:swap`
- `htmx:before:cleanup`
- `htmx:response:error`

Request and response state lives under `event.detail.ctx`, including `ctx.request`, `ctx.response.status`, `ctx.text`, and `ctx.target`. Do not read v2 `detail.xhr` or `requestConfig` fields.

Keep `hx-on` handlers short and local. Put reusable or stateful behavior in the project's JavaScript modules. If a component framework already owns a DOM subtree, avoid having htmx replace inside that subtree without an explicit integration boundary.

Client-owned charts, maps, canvas views, and editors may read safely encoded inert data islands from the returned HTML. Use the backend's HTML-safe JSON serializer rather than string interpolation. Keep initialization idempotent, scope cleanup to nodes actually being removed, and do not assume undocumented lifecycle ordering for partial or out-of-band updates.

## Handle navigation and history

Use `hx-boost` for progressively enhanced links/forms with page-navigation defaults. Use `hx-push-url` or `hx-replace-url` only when the resulting URL can be opened directly and the server can reconstruct the visible state.

htmx 4 does not keep the htmx 2 localStorage history cache. Back/forward restoration re-fetches content and swaps it into `<body>` or `[hx-history-elt]`. Test expired sessions, validation state, scroll/focus, and server rendering on restoration. Add a history extension only for a concrete offline or cache requirement.

## Load extensions deliberately

Include an htmx 4 extension script directly; do not add `hx-ext`. Verify that the extension version supports the installed htmx prerelease. This is especially important for SSE, WebSockets, Alpine integration, history caching, optimistic updates, and CSP behavior.

Registered extension hooks apply page-wide unless the extension activates through its own custom attributes. When replacing a v2 custom extension that used `hx-ext` for subtree scope, guard v4 hooks with a project-owned marker such as `data-my-extension`.

For custom extensions, use `htmx.registerExtension()` and htmx 4 hooks. Read the current extension documentation instead of adapting a v2 callback by name.

## Preserve security and caching

Use the backend's established CSRF, session, authorization, validation, escaping, and redirect mechanisms. Escape all user-controlled values before returning HTML. Avoid `js:` values, trigger filters, and inline `hx-on` where a strict CSP or untrusted content makes evaluation unsafe.

Do not trust `HX-Request`, `HX-Source`, or `HX-Target` as proof of identity, permission, or origin. Apply access control before rendering the fragment.

If a cacheable URL varies by `HX-Request`, send `Vary: HX-Request`. If it varies by other headers, include those too. Never let a fragment response poison a full-page cache entry.

## Debug and test

Debug from the contract outward:

1. Confirm the source event fires and only one request is sent.
2. Inspect method, URL, submitted values, CSRF data, and htmx headers.
3. Inspect response status and raw HTML.
4. Confirm the target exists before the swap and the response root fits it.
5. Check console warnings, duplicate IDs, invalid HTML, and lifecycle code.
6. For temporary tracing, enable `htmx.config.logAll = true`.

Test handlers for statuses, headers, and fragment shape. Use browser tests only where DOM ordering, focus, morphing, concurrency, history, or third-party initialization affects correctness.
