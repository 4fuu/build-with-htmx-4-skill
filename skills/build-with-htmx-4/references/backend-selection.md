# Backend Selection for a New htmx 4 Project

Use this workflow when a new project has no settled backend stack or when the user explicitly reopens the choice. It applies to the current skill revision and `htmx.org@4.0.0-beta6`. Runtime, framework, package, and hosting support can change; verify selected versions and the deployment target in current official documentation before scaffolding.

## Start from workspace evidence

Inspect before asking questions or proposing a stack:

- Project manifests and locks: `package.json`, npm/pnpm/Yarn locks, `bun.lock*`, `deno.json*`, `deno.lock`, `pyproject.toml`, Python locks, `go.mod`, `Gemfile`, `composer.json`, Maven or Gradle files, `.csproj`, `.sln`, `global.json`, and `Cargo.toml`.
- Runtime pins and tool configuration: `.nvmrc`, `.node-version`, `.tool-versions`, `mise.toml`, SDK files, compiler targets, and package-manager declarations.
- Application evidence: routes, handlers, templates, ORM models, authentication, sessions, queues, tests, linters, formatters, and adjacent projects in the same workspace.
- Operations evidence: `Dockerfile`, Compose, CI workflows, infrastructure files, process definitions, target OS and architecture, serverless or edge configuration, and the hosting provider's supported runtimes.
- External constraints: database and driver, vendor SDKs, native packages, filesystem access, uploads, long-lived connections, scheduled work, background workers, and organizational language or framework rules.

A globally installed runtime establishes local availability only. A manifest, lockfile, runtime pin, deployment file, or explicit user statement establishes a project constraint.

## Ask only unresolved questions

Ask one to three short questions before scaffolding. Combine related choices and omit anything already answered:

1. Which language, runtime, framework, or rendering syntax must be used or reused?
2. Where will it run: a long-lived server, container, serverless function, edge runtime, or named hosting platform?
3. Which initial features determine infrastructure or framework facilities: database, authentication, forms and validation, admin UI, uploads, scheduled tasks, background jobs, WebSockets, or other stated scope?

When the user has not named an exact htmx version, disclose that this skill targets prerelease `4.0.0-beta6` and obtain confirmation before creating a new project. Bundle this with another question when possible.

If neither the workspace nor the request narrows the language ecosystem, ask these questions before loading ecosystem details or listing concrete frameworks. After the answer, read only `backend-stacks-js.md` or `backend-stacks-other.md` as routed by `SKILL.md`.

If the user explicitly has no ecosystem preference, treat the task as a cross-ecosystem comparison. First collect deployment and facility requirements, then load both ecosystem references and compare only ecosystems that satisfy the hard constraints. Ask the user to choose an ecosystem before presenting its framework-level candidates unless the user delegated that choice.

If existing evidence narrows the ecosystem, present only matching candidates and then ask the user to select. If the user has already selected a stack, ask only about a detected conflict or a missing requirement that changes the scaffold.

## Present options as conditions

Use this shape:

| Option | Detected facts it reuses | Components it adds | Checks before use |
| --- | --- | --- | --- |
| Runtime + framework + renderer | Existing manifests, code, deployment, or team rule | Runtime, framework, renderer, data, auth, or job component not already present | Host support, package APIs, native modules, permissions, process model |

State measurable facts. Do not rank options or use developer-experience, popularity, ecosystem-quality, performance, or maintainability claims. Do not make performance claims without a benchmark matching the application's workload and deployment.

Any HTTP backend that can accept form requests and return HTML can serve htmx. The stack choice determines routing, rendering, validation, security, data access, background work, packaging, and deployment; it does not change htmx's HTML-over-HTTP contract.

## Apply objective rules

Apply hard constraints before user preferences:

1. Exclude options the deployment target cannot execute or package.
2. Preserve an existing application's language, framework, renderer, database layer, and conventions unless the user requests a change.
3. Exclude options that cannot use a required database driver, vendor SDK, native add-on, authentication system, long-lived connection, worker, filesystem API, or platform permission.
4. List which required facilities each candidate supplies and which require another package or service.
5. When server-side JSX and templates both satisfy the requirements, ask which syntax the user wants; do not infer it from TypeScript alone.
6. When several candidates remain, ask the user to select unless the user delegated selection.
7. For delegated selection, first choose the candidate that reuses more named project evidence: source, lockfile, runtime pin, deployment configuration, tests, data layer, authentication, or operational tooling. State the counted evidence.
8. If reuse is tied, count only facilities the user explicitly required: renderer, data access and migrations, authentication and sessions, forms and validation, admin UI, uploads, scheduled work, background jobs, and WebSockets. Count a facility as supplied only when the chosen official scaffold includes it without another package, license, service, or process; otherwise list the addition. Choose the candidate that supplies more required facilities, then the one with fewer listed additions. Show the count and name every facility and addition.
9. If that count is also tied, ask the user to break the tie. Delegation does not authorize an arbitrary choice.

If hard constraints exclude every candidate, stop before creating files. Show the conflicting facts and ask which constraint may change.

Convert terms such as “easy,” “fast,” or “maintainable” into a checkable condition or ask what outcome the user means. Examples include number of deployable processes, required framework facilities, cold-start limit, maximum artifact size, or a named package that must work.

## Check deployment-specific requirements

- For native dependencies, verify whether they are required or conditionally loaded, plus supported runtime, ABI, OS, and architecture.
- For uploads, identify persistent storage, size limits, request and streaming support, and retention requirements. Do not assume an instance filesystem is persistent.
- For scheduled work, distinguish a platform scheduler or function from a long-lived worker. Check time zone, maximum duration, concurrency, retries, and idempotency.
- For edge or serverless targets, check filesystem, process, socket, native-code, request-duration, and background-execution limits in the platform documentation.

## Record and verify the choice

Before creating files, summarize:

- selected language, runtime, framework, renderer, package manager, htmx version, and other version constraints;
- deployment target and process model;
- database, authentication, validation, session, job, upload, and asset choices that affect the scaffold;
- detected files and requirements used in the decision;
- compatibility checks still required.

Then scaffold with the selected runtime's current official command or documented minimum structure. Pin versions through the ecosystem's normal manifest and lockfile, run the generated tests or checks, and verify one full-page response plus one htmx fragment response.
