# Build with htmx 4 Skill

[![skills.sh](https://skills.sh/b/4fuu/build-with-htmx-4-skill)](https://skills.sh/4fuu/build-with-htmx-4-skill)

A version-specific [Agent Skill](https://agentskills.io/) for building, reviewing, debugging, and migrating server-driven web interfaces with [htmx 4](https://four.htmx.org/).

The repository is named **`build-with-htmx-4-skill`**; the installable skill ID is **`build-with-htmx-4`**.

## What it provides

- Version checks that keep htmx 4 syntax separate from htmx 2
- An evidence-based backend selection workflow for JavaScript, TypeScript, Python, Go, Ruby, PHP, Java/Kotlin, .NET/C#, and Rust
- Guidance for server-rendered architecture, HTML fragment endpoints, progressive enhancement, and shared partials
- htmx 4 patterns for forms, validation errors, multi-target updates, explicit inheritance, swaps, concurrency, events, history, and extensions
- Security, caching, debugging, and response-contract testing guidance
- A structured migration guide from htmx 2 to htmx 4

## Install with `npx skills`

Inspect the skills detected in this repository:

```bash
npx skills add 4fuu/build-with-htmx-4-skill --list
```

Install `build-with-htmx-4` for detected agents:

```bash
npx skills add 4fuu/build-with-htmx-4-skill --skill build-with-htmx-4
```

Install non-interactively for [Pi](https://github.com/earendil-works/pi):

```bash
npx skills add 4fuu/build-with-htmx-4-skill --skill build-with-htmx-4 --agent pi -y
```

Install globally for Pi:

```bash
npx skills add 4fuu/build-with-htmx-4-skill --skill build-with-htmx-4 --agent pi --global -y
```

A direct repository path is also supported:

```bash
npx skills add https://github.com/4fuu/build-with-htmx-4-skill/tree/main/skills/build-with-htmx-4
```

This repository uses the conventional `skills/build-with-htmx-4/SKILL.md` layout and valid `name`/`description` frontmatter, so the `skills` CLI can discover it without extra manifests or npm packaging.

## When it triggers

The skill is intended for tasks involving:

- htmx 4 or `htmx.org@4.x`
- New server-rendered htmx projects and backend/template selection
- htmx markup, handlers, HTML fragments, swaps, forms, and validation
- Multi-target updates, events, extensions, history, caching, or debugging
- Migration from htmx 2 to htmx 4

For example:

- “Create an htmx 4 form that returns validation errors to a separate target.”
- “Choose a backend and template stack for this new htmx project.”
- “Migrate these htmx 2 attributes and event handlers to htmx 4.”
- “Debug why this fragment response swaps into the wrong DOM boundary.”

## Repository layout

```text
build-with-htmx-4-skill/
├── LICENSE
├── README.md
└── skills/
    └── build-with-htmx-4/
        ├── LICENSE
        ├── SKILL.md
        ├── agents/
        │   └── openai.yaml
        ├── assets/
        │   └── icon.svg
        └── references/
            ├── architecture.md
            ├── backend-selection.md
            ├── backend-stacks-js.md
            ├── backend-stacks-other.md
            ├── htmx-4-patterns.md
            └── migrate-from-htmx-2.md
```

`SKILL.md` contains the core workflow. Longer architecture, implementation, stack-selection, and migration guides are loaded on demand from `references/`.

## Versioning

This revision targets **`htmx.org@4.0.0-beta6`**. htmx 4 is a prerelease, so syntax and behavior may change. The skill directs agents to inspect the installed version and consult the matching official documentation instead of silently upgrading, downgrading, or applying htmx 2 behavior.

Official htmx 4 references:

- https://four.htmx.org/docs
- https://four.htmx.org/reference
- https://four.htmx.org/docs#migration

## Requirements

- No dependency is required to read or install the skill.
- Node.js/npm is required only for the `npx skills` installation commands.
- The selected backend runtime and project tooling are required when carrying out implementation or verification work.

## License

MIT. See [LICENSE](LICENSE).
