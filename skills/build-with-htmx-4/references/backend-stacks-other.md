# Python, Go, Ruby, PHP, JVM, .NET, and Rust Backend Stacks

This reference describes capabilities checked for the current skill revision. Runtime and framework support changes; inspect the selected version and current official documentation before scaffolding.

## Contents

- [Ecosystem matrix](#ecosystem-matrix)
- [Python](#python)
- [Go](#go)
- [Ruby](#ruby)
- [PHP](#php)
- [Java and Kotlin](#java-and-kotlin)
- [.NET and C#](#net-and-c)
- [Rust](#rust)

## Ecosystem matrix

| Ecosystem | Framework choices | HTML rendering choices | Project and deployment facts to check |
| --- | --- | --- | --- |
| Python | Django, Flask, FastAPI | Django templates, Jinja, or another ASGI/WSGI-compatible renderer | Python version, WSGI or ASGI server, dependency lock, sync or async database driver, worker model |
| Go | `net/http`, Chi, Echo, Fiber | `html/template`, templ, or another Go renderer | Go version, code-generation step, CGO and native libraries, target OS and architecture, serverless adapter |
| Ruby | Rails, Sinatra, Roda | Action View, ERB, Haml, Slim | Ruby version, Bundler lock, Rack server, Active Record or another data layer, job process, host support |
| PHP | Laravel, Symfony, framework components | Blade, Twig, native PHP templates | PHP version and extensions, Composer lock, PHP-FPM or long-lived process model, web-server configuration, workers |
| Java or Kotlin | Spring Boot MVC, Ktor | Thymeleaf, FreeMarker, Mustache, JTE, Pebble, or Kotlin HTML DSL | JDK target, Maven or Gradle, servlet or reactive stack, JVM memory, native-image requirements, host support |
| .NET or C# | ASP.NET Core Razor Pages, MVC, Minimal APIs | Razor views or pages; a selected renderer for Minimal APIs | Target framework, installed SDK, Kestrel, IIS or container hosting, self-contained or framework-dependent publish mode |
| Rust | Axum, Actix Web, Rocket | Askama, Maud, Tera | Rust toolchain, async runtime, template compile or runtime behavior, database and TLS crate features, compilation target |

## Python

- Django supplies routing, templates, forms and validation, ORM, authentication, sessions, migrations, and an admin application. Check which supplied facilities the project will use.
- Flask supplies routing and Jinja integration. Select database, forms, authentication, migrations, and background-job packages when required.
- FastAPI runs on ASGI and can render Jinja templates through Starlette. Define HTML responses and templates explicitly for htmx fragments.
- Documentation: `https://docs.djangoproject.com/`, `https://flask.palletsprojects.com/`, `https://fastapi.tiangolo.com/advanced/templates/`.

## Go

- The standard library combines `net/http` with context-aware `html/template` escaping.
- Routers and frameworks such as Chi, Echo, and Fiber change routing and middleware APIs. Confirm whether existing middleware targets the standard `net/http` interfaces.
- templ generates Go code from `.templ` files. Include its generation command in local development and CI when selected.
- Documentation: `https://pkg.go.dev/net/http`, `https://pkg.go.dev/html/template`, `https://templ.guide/`.

## Ruby

- Rails includes routing, controllers, Action View, forms, validation, Active Record, sessions, security helpers, migrations, and jobs.
- Sinatra and Roda provide routing and response handling; add the chosen renderer and application facilities explicitly.
- Rails Turbo and htmx can both intercept navigation, form submission, and DOM updates. If both are present, define which system owns each link, form, and DOM region.
- Documentation: `https://guides.rubyonrails.org/`, `https://sinatrarb.com/documentation.html`, `https://roda.jeremyevans.net/`.

## PHP

- Laravel includes Blade, validation, Eloquent, authentication facilities, sessions, queues, and migrations through its framework and first-party packages.
- Symfony provides Twig integration, forms, validation, security, sessions, Messenger, and database integrations as separate components and bundles.
- PHP-FPM handles requests through workers started by a process manager. Long-lived PHP runtimes retain process state between requests. Select configuration and lifecycle handling for the actual host model.
- Documentation: `https://laravel.com/docs`, `https://symfony.com/doc/current/`.

## Java and Kotlin

- Spring Boot MVC can render Thymeleaf, FreeMarker, Mustache, or other configured view technologies. Security, validation, data access, and jobs are supplied through selected Spring modules and dependencies.
- Ktor supports an HTML DSL and template plugins including Thymeleaf, FreeMarker, Mustache, Pebble, and JTE. Data access and authentication use selected plugins or libraries.
- Record whether the application uses servlet MVC, a reactive stack, or Ktor's engine APIs because middleware and streaming behavior differ.
- Documentation: `https://docs.spring.io/spring-boot/`, `https://docs.spring.io/spring-framework/reference/web/webmvc-view.html`, `https://ktor.io/docs/server-templating.html`.

## .NET and C#

- Razor Pages provides page-focused handlers and Razor views. MVC separates controllers and views. Both use ASP.NET Core model binding, validation, dependency injection, middleware, and optional Identity.
- Minimal APIs define routes without the Razor Pages or MVC view pipeline. Add and test a rendering mechanism when they return HTML fragments.
- Record the target framework and publish mode because the host may require an installed runtime or a self-contained deployment artifact.
- Documentation: `https://learn.microsoft.com/aspnet/core/`.

## Rust

- Axum uses Tower middleware and extractors. Actix Web and Rocket use their own handler, extraction, and middleware APIs.
- Askama compiles Jinja-like templates into Rust code. Maud constructs HTML with Rust macros. Tera loads templates through its template engine. Their build and reload behavior differs.
- Database clients, migrations, forms, sessions, authentication, and jobs are selected as crates or application modules.
- Documentation: `https://docs.rs/axum/`, `https://actix.rs/docs/`, `https://rocket.rs/`, `https://askama.rs/`, `https://maud.lambda.xyz/`, `https://keats.github.io/tera/`.
