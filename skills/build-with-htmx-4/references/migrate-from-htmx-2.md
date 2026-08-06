# Migrate from htmx 2 to htmx 4

This guide targets migration to `htmx.org@4.0.0-beta6`. htmx 4 is a prerelease, so re-check the official migration guide and version-matched upgrade checker when upgrading to another release.

## Contents

1. [Choose a migration mode](#choose-a-migration-mode)
2. [Inventory before editing](#inventory-before-editing)
3. [Migrate in a safe order](#migrate-in-a-safe-order)
4. [Update behavior changes](#update-behavior-changes)
5. [Rename or replace attributes](#rename-or-replace-attributes)
6. [Update events and request context](#update-events-and-request-context)
7. [Update headers, configuration, and APIs](#update-headers-configuration-and-apis)
8. [Migrate extensions](#migrate-extensions)
9. [Remove compatibility mode](#remove-compatibility-mode)

## Choose a migration mode

Prefer an incremental migration for a production application with many templates or extensions. A small, well-tested application can migrate directly.

Two temporary compatibility choices restore the highest-impact v2 behavior:

```html
<meta name="htmx-config"
      content='{"implicitInheritance": true,
                "noSwap": [204, 304, "4xx", "5xx"]}'>
<script src="/static/htmx-4.0.0-beta6.min.js"></script>
```

Or load the version-matched `htmx-2-compat` extension after htmx. It also restores old event names and legacy `hx-ext` activation. Treat either choice as a bridge with an explicit removal plan, not the finished migration.

Compatibility does not restore `XMLHttpRequest`; code using `detail.xhr` must migrate to `detail.ctx`. Do not assume an arbitrary v2 third-party or custom extension works until its behavior is tested against the installed compat and htmx versions.

Do not remove compatibility settings while untouched v2 templates or scripts still depend on them.

## Inventory before editing

Pin the intended htmx 4 version and run its checker:

```bash
npx htmx.org@4.0.0-beta6 upgrade-check -- ./path/to/project
```

Add `--ext .vue` or another extension when templates use a non-default suffix. Save the report as a migration checklist and also search for:

- all htmx script and extension versions;
- global configuration and response-header middleware;
- `hx-*` attributes in templates and generated markup;
- `htmx:*` listeners and `hx-on` attributes;
- `detail.xhr`, `requestConfig`, and removed helper APIs;
- custom extensions, custom swaps, history assumptions, and browser tests.

Do not rely on search alone: attributes may be produced by template helpers or backend components.

## Migrate in a safe order

1. Pin htmx 4 and add temporary compatibility behavior when needed.
2. Run the upgrade checker and establish tests around affected flows.
3. Rename the conflicting disable attributes in the required order.
4. Make inheritance explicit and design error-response swaps.
5. Replace removed attributes and trigger queuing.
6. Update events, context fields, headers, configuration, and JavaScript APIs.
7. Upgrade built-in and custom extensions.
8. Test navigation, concurrency, multi-target updates, and long requests.
9. Remove compatibility behavior one slice at a time.

Migrate by feature or route when possible. Keep each change small enough that a failed target, status, or event can be traced to one contract.

## Update behavior changes

| htmx 2 behavior | htmx 4 beta6 behavior | Migration action |
| --- | --- | --- |
| Requests use XHR | Requests use `fetch()` | Remove XHR assumptions; use `detail.ctx` and fetch-compatible interception |
| Attributes commonly inherit | Inheritance is explicit | Add `:inherited`; use `:append` where a child extends a value |
| `4xx`/`5xx` normally do not swap | Every status except `204`/`304` swaps | Return target-safe error HTML or configure `hx-status`/`noSwap` |
| `hx-delete` can include form values | DELETE excludes enclosing form values | Add `hx-include="closest form"` only where required |
| History can use localStorage cache | Restoration re-fetches | Test server reconstruction and back/forward behavior |
| OOB updates happen before main swap | Main swap happens first; OOB/partials follow in document order | Remove ordering dependencies |
| `hx-trigger` supports `queue:*` | Trigger queue modifier is removed | Use `hx-sync="this:queue ..."` |
| Default timeout is unlimited | Default timeout is 60 seconds | Decide whether long endpoints need config or redesign |
| Extensions are selected with `hx-ext` | Extension scripts register when loaded | Include scripts directly and remove `hx-ext` |

Make URLs and server rendering sufficient to restore navigation. If local snapshots are a real requirement, beta6 provides the `hx-history-cache` extension, which stores pages in `sessionStorage`. It is not equivalent to v2 localStorage across tabs or browser restarts; verify its current configuration, exclusion, and lifecycle-event documentation before adopting it.

Example inheritance change:

```html
<!-- v2 -->
<section hx-confirm="Delete this item?">
  <button hx-delete="/items/42">Delete</button>
</section>

<!-- v4 -->
<section hx-confirm:inherited="Delete this item?">
  <button hx-delete="/items/42">Delete</button>
</section>
```

Example queue change:

```html
<!-- v2 -->
<button hx-post="/vote" hx-trigger="click queue:all">Vote</button>

<!-- v4 -->
<button hx-post="/vote" hx-sync="this:queue all">Vote</button>
```

`this` preserves a queue local to one stable source. If several controls share a race domain or the source is replaced by a swap, point `hx-sync` at a stable common container and re-check the intended `first`, `last`, or `all` behavior.

## Rename or replace attributes

Rename the disable pair in this order to avoid changing meaning mid-migration:

1. v2 `hx-disable` -> v4 `hx-ignore` (skip htmx processing).
2. v2 `hx-disabled-elt` -> v4 `hx-disable` (disable selected controls during a request).

That rename order applies to v4-only output. During a dual-version rollout, branch templates or helpers by the selected htmx resource bundle; the same `hx-disable` attribute cannot safely carry its v2 and v4 meanings in shared HTML.

Common removals:

| htmx 2 | htmx 4 beta6 | Notes |
| --- | --- | --- |
| `hx-vars` | `hx-vals="js:..."` | Prefer static values when evaluation is unnecessary |
| `hx-params` | `htmx:config:request` | Modify request values in the event context |
| `hx-prompt` | `hx-prompt` extension or local JS | Load only if prompts are still required |
| `hx-ext` | none | Include extension scripts directly |
| `hx-inherit`, `hx-disinherit` | explicit `:inherited` | Default is no inheritance |
| `hx-request` | `hx-config` | Re-check current option names |
| `hx-history` | none | There is no core localStorage history cache |

Do not mechanically add `:inherited` everywhere. Put attributes directly on the source when region-level inheritance does not express a useful policy.

## Update events and request context

Event names now use `htmx:<phase>:<action>`:

| htmx 2 | htmx 4 beta6 |
| --- | --- |
| `htmx:configRequest` | `htmx:config:request` |
| `htmx:beforeRequest` | `htmx:before:request` |
| `htmx:afterRequest` | `htmx:after:request` |
| `htmx:beforeSwap` | `htmx:before:swap` |
| `htmx:afterSwap` | `htmx:after:swap` |
| `htmx:afterSettle` | `htmx:after:settle` |
| `htmx:beforeCleanupElement` | `htmx:before:cleanup` |
| response-related error events | `htmx:response:error` |

Update both JavaScript listeners and `hx-on:*` attributes. Do not assume every old error event has a one-to-one rename; most consolidate under `htmx:error`, while HTTP response errors use `htmx:response:error`.

Replace XHR-specific access:

```js
// v2
const status = event.detail.xhr.status

// v4
const status = event.detail.ctx.response.status
```

The v4 context contains `ctx.request`, `ctx.response`, `ctx.text`, and `ctx.target`. Modify request values in `htmx:config:request`; transform received text during `htmx:after:request` before fragment creation. Verify the current event reference before changing lifecycle-sensitive code.

## Update headers, configuration, and APIs

Common request-header changes:

| htmx 2 | htmx 4 beta6 |
| --- | --- |
| `HX-Trigger` request header | `HX-Source`, formatted as `tag#id` |
| `HX-Trigger-Name` | removed; use `HX-Source` plus submitted values as needed |
| `HX-Target` ID value | `HX-Target` formatted as `tag#id` |
| no equivalent | `HX-Request-Type: partial|full` |

The response `HX-Trigger` remains, but `HX-Trigger-After-Swap` and `HX-Trigger-After-Settle` are removed. Move timing-sensitive client work to explicit lifecycle listeners.

Configuration keys and defaults changed beyond the high-impact items listed here. Compare every project override with the current reference; delete obsolete overrides rather than guessing their replacements.

Prefer native DOM methods where v2 convenience APIs were removed. Update `htmx.defineExtension()` to `htmx.registerExtension()`.

## Migrate extensions

For packaged extensions, use a release that explicitly supports the installed htmx 4 version. Load its script directly and remove corresponding `hx-ext` attributes.

V4 extension hooks register page-wide unless the extension activates through its own attributes. If a v2 custom extension relied on `hx-ext` to limit behavior to one subtree, add a project-owned marker and return early from each hook outside that scope before removing `hx-ext`.

For custom extensions:

| htmx 2 callback | htmx 4 hook |
| --- | --- |
| `onEvent(name, evt)` | individual hooks such as `htmx_before_swap` |
| `transformResponse(...)` | `htmx_after_request`; modify `detail.ctx.text` |
| `encodeParameters(...)` | `htmx_before_request`; modify the final request body |
| `getSelectors()` | `htmx_after_init` |
| `isInlineSwap()` / `handleSwap()` | `handle_swap` |

Hook method names replace event colons with underscores. Test custom swap styles in both primary and out-of-band responses; non-outer OOB swaps strip their wrapper, so wrapper-preserving custom swaps should use an `outer...` name.

Do not port an extension by search-and-replace alone. Verify registration, cancellation, request mutation, response transformation, swap return values, and cleanup behavior against the current extension guide.

## Remove compatibility mode

Remove compatibility one feature slice at a time. For each slice, verify:

- inherited attributes still reach only intended descendants;
- `422`, `4xx`, and `5xx` responses render in safe targets;
- no listener depends on an old event name or `detail.xhr`;
- history restoration works after a fresh server request;
- multi-target swaps do not depend on v2 ordering;
- duplicate actions and validation requests follow the intended `hx-sync` strategy;
- all loaded extensions are native htmx 4 versions.

Re-run the version-matched upgrade checker after removing compatibility behavior, then run handler/template and browser tests. Keep the migration incomplete rather than silently preserving unexplained compatibility settings.
