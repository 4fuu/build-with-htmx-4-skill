---
name: build-with-htmx-4
description: Create, select a backend for, modify, review, debug, or migrate server-driven web interfaces using htmx 4.0.0-beta6. Use whenever a project uses htmx 4 or the user asks for a new htmx 4 project, backend or template selection, htmx markup, HTML fragment endpoints, swaps, forms, validation errors, multi-target updates, events, extensions, history, or migration from htmx 2. Do not apply htmx 4 syntax to work that explicitly remains on htmx 2.
---

# Build with htmx 4

## Confirm the version

These instructions describe `htmx.org@4.0.0-beta6`. htmx 4 is a prerelease; later versions can change syntax and behavior.

Inspect the package, lockfile, vendored script, or CDN URL before editing. Do not silently upgrade or downgrade. When the project uses another version, consult its official documentation before applying version-specific guidance from this skill:

- `https://four.htmx.org/docs`
- `https://four.htmx.org/reference`
- `https://four.htmx.org/docs#migration`
- `https://four.htmx.org/llms.txt`
- `https://four.htmx.org/llms-full.txt`

## Read the relevant layer

Load only the references needed for the task:

- Read [references/backend-selection.md](references/backend-selection.md) before choosing or scaffolding the backend, runtime, framework, or template system for a new htmx 4 project.
- After the environment or user narrows the ecosystem, read [references/backend-stacks-js.md](references/backend-stacks-js.md) for Node.js, Bun, or Deno, or read [references/backend-stacks-other.md](references/backend-stacks-other.md) for Python, Go, Ruby, PHP, Java/Kotlin, .NET/C#, or Rust. Do not load both unless the user is comparing those groups.
- Read [references/architecture.md](references/architecture.md) before designing a new interaction, choosing the server/client boundary, reviewing architecture, or translating patterns from React, Vue, Svelte, or a JSON API.
- Read [references/htmx-4-patterns.md](references/htmx-4-patterns.md) when writing, reviewing, or debugging htmx 4 markup, handlers, fragments, errors, multi-target responses, events, history, or extensions.
- Read [references/migrate-from-htmx-2.md](references/migrate-from-htmx-2.md) for htmx 2 code, compatibility mode, upgrade planning, or legacy attributes, events, headers, configuration, APIs, and extensions.

## Select a stack for a new project

Apply this section only when creating a project or when the task explicitly reopens its backend choice.

1. Inspect the workspace for manifests, lockfiles, runtime pins, source files, containers, CI, deployment configuration, databases, and existing conventions.
2. Read `references/backend-selection.md`.
3. Ask one to three short questions about unresolved requirements. Do not repeat choices already stated in the request. When neither the workspace nor the request narrows the ecosystem, ask before presenting concrete frameworks.
4. Read only the matching ecosystem reference, then relate each remaining option to detected facts and stated requirements. Report required additions and compatibility checks without rankings or subjective labels.
5. Do not scaffold until the user selects an option or explicitly delegates the selection. If selection is delegated, apply the objective rules in the reference and state which detected facts determined the result. Stop and ask when hard constraints exclude every option or when the objective rules leave a tie.

Treat an executable found on `PATH` as available tooling, not as the user's preferred project stack. Do not replace an existing project stack merely because another runtime is installed.

## Follow the application

Keep the existing backend, template engine, routing, styling, JavaScript, and test conventions unless the task requests an architectural change. Inspect adjacent handlers and templates before inventing a new response or component pattern.

Keep durable business state, validation, authorization, and rendering decisions on the server. Use htmx for server interactions and return HTML the browser can swap directly. Use ordinary browser JavaScript or the project's client framework for local-only state and client-heavy interactions.

Do not force drag-and-drop state machines, undo stacks, offline queues, canvas interactions, or similar behavior into htmx attributes. Keep htmx at the HTTP boundary when such behavior also needs server synchronization.

## Define the interaction before coding

For every interaction, identify:

1. Source element and trigger.
2. HTTP method, URL, and submitted values.
3. Primary target and swap strategy.
4. Success response status and exact HTML shape.
5. Expected error statuses and their HTML targets.
6. Additional regions updated by the same response.
7. Loading, repeat-click, race, and history behavior when relevant.

Make the server response match the selected DOM boundary. Reuse partial templates between full-page and htmx responses. Add stable IDs only where targeting, preservation, history, or multi-target updates require them.

## Implement the whole contract

Change the triggering markup, server handler, shared template fragments, and focused tests together. Preserve normal links and forms when progressive enhancement is useful.

Treat htmx requests as ordinary application requests. Preserve authentication, authorization, CSRF protection, server validation, output escaping, redirects, caching, and database constraints. Do not treat htmx headers or client-side disabling as security or integrity controls.

When the same cacheable URL returns a full page and a fragment based on `HX-Request`, send `Vary: HX-Request`.

## Verify observable behavior

Run the project's existing tests, then verify the states the change can enter:

- success, validation error, authorization error, empty state, and server failure;
- repeated actions and out-of-order responses where applicable;
- the exact target, swap, focus, indicator, and additional-region updates;
- full-page fallback and back/forward navigation when supported;
- valid HTML, unique purposeful IDs, and no invalid nested forms.

Prefer handler/template tests for response contracts and browser tests for DOM ordering, morphing, focus, third-party widget lifecycle, or history behavior. Test application behavior rather than htmx internals.
