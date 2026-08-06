# JavaScript and TypeScript Backend Stacks

This reference describes capabilities checked for the current skill revision. Runtime and framework support changes; inspect the selected version and current official documentation before scaffolding.

## Runtime matrix

| Runtime | Project evidence | HTTP and framework choices | HTML rendering choices | Compatibility checks |
| --- | --- | --- | --- | --- |
| Node.js | `package.json`, Node lockfiles, Node version pins, Node-based deployment | `node:http`, Express, Fastify, Hono; AdonisJS or NestJS when their modules are required | EJS, Nunjucks, Pug, Handlebars, Eta, framework templates, or server-side JSX | Host Node version, module format, package-manager lock, native add-ons, serverless adapter |
| Bun | `package.json`, `bun.lock*`, Bun runtime pin or Bun deployment | `Bun.serve`, Hono, Elysia; Node-targeted frameworks after compatibility verification | Template packages, Hono JSX, or another server-side JSX renderer | Bun's current Node API coverage, native add-ons, postinstall scripts, stream and process APIs, host support |
| Deno | `deno.json*`, `deno.lock`, import map, JSR or npm imports, Deno deployment | `Deno.serve`, Hono, Oak, Fresh | Eta, Hono JSX, Preact or React server rendering, Fresh JSX | Network, environment and filesystem permissions; npm and Node API compatibility; native add-ons; host support |

## Node.js

- Node.js uses Node APIs and the npm package ecosystem.

| Candidate | Documented framework boundary | Rendering path |
| --- | --- | --- |
| `node:http` | Node HTTP server API; routing, middleware, validation, and application facilities remain application choices | Add a renderer or generate escaped HTML |
| Express | Routing and middleware; view engine configured by the application | EJS, Pug, Handlebars, or another compatible view engine |
| Fastify | Routing, hooks, schemas, and plugins | `@fastify/view` with a supported template engine |
| Hono on Node | Hono routing and middleware through its Node adapter | Built-in JSX renderer or another response renderer |
| Nest MVC | Modules, dependency injection, controllers, and Express or Fastify adapter | Adapter view engine and `@Render` or response rendering |
| AdonisJS | Routing plus official packages for Edge, validation, sessions, authentication, database access, and jobs | Edge templates |

Check the project's Node version, ESM or CommonJS mode, lockfile, package manager, native add-ons, deployment adapter, and renderer before choosing among these frameworks.

## Bun

- Bun includes a runtime, package manager, test runner, transpiler, bundler, and `Bun.serve`.
- `Bun.serve` supplies the runtime HTTP server. Hono supplies routing, middleware, and JSX. Elysia supplies routing, lifecycle hooks, schemas, and validation. Node-targeted frameworks can be candidates after their dependencies and runtime APIs are verified on the selected Bun version.
- Bun's Node compatibility is documented as an ongoing implementation. Test dependencies that use Node internals, native code, postinstall scripts, streams, or process APIs.
- A `package.json` without a Bun lockfile or runtime pin does not by itself establish Bun as the project runtime.

## Deno

- Deno supplies web APIs, `Deno.serve`, JSR and npm package access, `node:` imports, `deno.json` configuration, lockfiles, and explicit permissions.
- `Deno.serve` supplies the runtime HTTP server. Hono supplies routing, middleware, and JSX. Oak supplies a middleware framework and router. Fresh supplies file-based server routes, JSX rendering, and optional islands. Eta and server-side Preact or React are other rendering choices.
- npm and Node API compatibility does not mean every Node package, native add-on, or tool works unchanged. Check required permissions and the target host.
- If Fresh islands and htmx both update a page, assign each DOM region to one system and avoid concurrent ownership of the same subtree.

## Cross-runtime Hono projects

Hono supports Node.js, Bun, and Deno. A shared Hono application still needs the runtime-specific entry point or adapter, scripts, lockfile, permissions, tests, and deployment configuration. Do not infer portability from framework source alone; run the selected runtime's tests.

## Official documentation

- Node.js: `https://nodejs.org/docs/latest/api/`
- Express templates: `https://expressjs.com/en/guide/using-template-engines.html`
- Fastify ecosystem: `https://fastify.dev/docs/latest/Guides/Ecosystem/`
- Hono runtimes and JSX: `https://hono.dev/docs/`
- AdonisJS hypermedia and Edge: `https://docs.adonisjs.com/stacks-and-starter-kits`
- Nest MVC: `https://docs.nestjs.com/techniques/mvc`
- Bun runtime and compatibility: `https://bun.sh/docs` and `https://bun.sh/docs/runtime/nodejs-compat`
- Elysia: `https://elysiajs.com/`
- Deno runtime, packages, and Node compatibility: `https://docs.deno.com/runtime/`
- Oak: `https://jsr.io/@oak/oak`
- Fresh: `https://fresh.deno.dev/`
