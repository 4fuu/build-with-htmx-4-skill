# htmx Architecture and Mental Model

Use this reference to decide where htmx belongs and what each endpoint should return. Version-specific syntax belongs in `htmx-4-patterns.md`.

## Reset the default frontend model

htmx extends links and forms: an element issues an HTTP request and the server's HTML response replaces a selected DOM region.

| Concern | Component SPA or JSON client | htmx application |
| --- | --- | --- |
| Response contract | Data, usually JSON | Rendered HTML matching a target |
| Render owner | Browser components | Server templates |
| Durable state | Often mirrored in a client store | Server/database remains authoritative |
| UI update | Change state, then re-render | Replace, append, delete, or morph a DOM region |
| Navigation | Client router often owns it | Normal URLs and browser history remain primary |
| Reuse unit | Client component | Server partial plus handler contract |

Do not add a JSON endpoint, client cache, or browser-side template solely to feed an htmx view. Keep an existing JSON API when other clients need it, but let the htmx route render HTML.

## Choose the boundary

Use htmx when an interaction naturally means “send input to the server, then render the server's new representation,” including CRUD, search, filtering, pagination, validation, moderation, and server-owned workflows.

Use ordinary JavaScript or the project's client framework for ephemeral behavior that should not require a server round trip: disclosure widgets, tabs, keyboard interactions, drag previews, rich editors, canvas, undo stacks, or offline queues.

For mixed interactions, keep local behavior local and use htmx only at persistence or synchronization points. Do not maintain the same business state independently in a client store and server-rendered HTML.

A client-owned chart, map, canvas, or editor may need structured input. Embed a safely serialized inert data island, such as `<script type="application/json">`, in the returned HTML and let the local module consume it. This remains an HTML response contract; it does not require a separate JSON endpoint or client-side template for the surrounding interface.

## Design one request-to-DOM contract

Write down this contract before choosing attributes:

| Part | Question |
| --- | --- |
| Source | Which element and user event initiate the request? |
| Request | Which URL, method, values, credentials, and CSRF data are sent? |
| Primary response | Which status and exact HTML boundary represent success? |
| Primary update | Which element is targeted, and is its inside, outside, or position changed? |
| Errors | Which expected statuses return user-safe HTML, and where should it render? |
| Side effects | Which other visible regions become stale after success? |
| Concurrency | Can two valid requests race or duplicate a durable action? |
| Navigation | Should the URL change, and can the page be restored by re-fetching it? |

The response root must fit the swap. `outerHTML` normally returns a replacement for the target itself; `innerHTML` returns its children; `beforeend` returns siblings valid inside the target container.

## Prefer one canonical URL and shared partials

For progressive enhancement, use a normal form or link plus htmx behavior:

```html
<form action="/items" method="post"
      hx-action="/items"
      hx-target="#item-list"
      hx-swap="beforeend">
  <input name="title" required>
  <button>Create</button>
</form>

<ul id="item-list">...</ul>
```

The normal POST can redirect to a full page. The htmx POST can return the rendered `<li>`. Prefer rendering both from the same item partial rather than maintaining duplicate markup.

On GET routes, the same URL may return the full layout without `HX-Request` and a fragment with it. If that response is cacheable, include `Vary: HX-Request`. Separate fragment URLs are acceptable when the application already uses them or the response contracts genuinely differ.

## Let the server settle business truth

After a write, return HTML rendered from the committed result, including normalized values, permissions, counts, and validation messages. Back important operations with transactions, uniqueness constraints, optimistic locking, or idempotency as appropriate.

When a write triggers retryable external work, dispatch it after a successful commit through the project's outbox or job mechanism and give the operation a durable idempotency key.

`hx-disable` and `hx-sync` improve interaction behavior but cannot prevent malicious requests, cross-tab duplicates, retries, or concurrent clients.

## Keep response topology simple

Prefer one primary target. Add out-of-band or partial updates only for regions that truly become stale, such as a count, flash message, or aggregate total. If many unrelated regions change on every request, reconsider the page boundary or return a larger coherent fragment.

Stable IDs are interface addresses. Add them to durable targets, preserved nodes, and ID-based out-of-band updates; avoid assigning IDs to every element.

## Avoid imported SPA habits

- Do not convert server HTML to JSON and reconstruct it in `hx-on`.
- Do not create a global client state store to coordinate server-rendered fragments.
- Do not hide substantial application logic in long inline event handlers.
- Do not use an htmx request when a normal link, native form, or local DOM action already solves the problem.
- Do not assume swapped DOM retains third-party widget instances; initialize and clean them up around htmx lifecycle events.
- Make widget initialization idempotent so restoration or repeated swaps cannot attach duplicate listeners.
- Do not return a full layout into a small target unless `hx-select` intentionally extracts the required region.

## Review the design

Before implementation, confirm that the server can render every success and expected-error state, the response boundary matches the swap, all stale regions are accounted for, local state has an explicit owner, and the flow remains understandable without reconstructing hidden client state.
