# GitHub Copilot Instructions

## Fork Attribution

This project (**PondManager**) is a fork of [MCSManager](https://github.com/MCSManager/MCSManager), licensed under the [Apache License 2.0](../LICENSE).

All fork-specific attributions are maintained in `PondManager/FORK_ATTRIBUTIONS.md`.

All original copyright notices, licence terms, and attribution from MCSManager are retained unmodified.

---

## Project Overview

PondManager is a fast-deploying, distributed, multi-user web-based management panel for Minecraft, Steam, and other game servers, built on top of MCSManager.

The codebase is split into three sub-projects:

| Sub-project | Path | Responsibility |
|---|---|---|
| Web backend | `panel/src/app/**` | User management, node connections, authentication, REST API |
| Daemon / node worker | `daemon/src/**` | Instance processes, containers, files, terminal management |
| Web frontend | `frontend/src/**` | Vue 3 UI; communicates with backend and, for performance-sensitive features, directly with the daemon |

---

## General Coding Rules

- **Minimal changes** — Prefer small, focused edits. Before adding new logic, check existing `hooks`, `services`, `stores`, and `utils` in the relevant sub-project and reuse them when possible.
- **Language** — All code and comments must be written in **English**.
- **No hardcoded user-facing strings** — Do not hardcode UI labels, messages, or error strings. Always use the project i18n flow (backend log messages are the only accepted exception).
- **Design quality** — Aim for high cohesion, low coupling, and reusable code.

---

## i18n Conventions

- **Frontend (Vue)**: use `t()` from `@/lang/i18n` for all translatable text.
- **Backend / Daemon**: use `$t()` (e.g. from `daemon/src/i18n/index.ts`) for all user-facing text and error messages.
- **Source language**: add and maintain source strings as short, correct English in `languages/en_us.json`. Other locales (e.g. `zh_CN`, `zh_TW`) are separate translations.

### Parameterised strings

Keys for dynamic messages use the `TXT_CODE_` prefix. Frontend and backend use **different** placeholder syntax:

| Context | Placeholder syntax |
|---|---|
| Frontend | Single braces: `{name}` |
| Backend / Daemon | Double braces: `{{uuid}}`, `{{err}}` |

```json
{
  "TXT_CODE_FILE_ERROR": "File {name} error!",
  "TXT_CODE_INSTANCE_ERROR": "Exception instance {{uuid}}: {{err}}"
}
```

```vue
<!-- Frontend usage -->
<template>{{ t("TXT_CODE_FILE_ERROR", { name: props.fileName }) }}</template>
```

```ts
// Backend / Daemon usage
const msg = $t("TXT_CODE_INSTANCE_ERROR", { uuid: instance.instanceUuid, err });
```

---

## Backend (Daemon & Panel) Conventions

Applies to `daemon/src/**/*.ts` and `panel/src/app/**/*.ts`.

### Directory / Layering

Folder names under `daemon/src/*` and `panel/src/app/*` express architectural layers (routes, middleware, services, instances, etc.). Place new backend logic in the appropriate layer and keep responsibilities clearly separated.

### Logging & Exceptions

- Use the project **logger** — never `console.*` directly. Choose severity (`info`, `error`, etc.) by context.
- For external resources (files, network, containers, shell):
  - Validate inputs and boundaries before acting.
  - On failure, log relevant context, then either rethrow or return a clear typed result the caller can handle.
  - **Never silently swallow exceptions.**

### Security

- For container and command execution: strictly parse and validate configuration (length, format, allowed values). Never pass unvalidated input into shell command arguments or container config fields.
- When reading files or running commands whose arguments originate from the frontend: validate paths, IDs, and other inputs for security risks.

### Memory & Resource Management

- When introducing long-lived structures (`Map`, queues, buffers, arrays, streams): add corresponding cleanup/release logic.
- Avoid unbounded growth — clear maps, trim log buffers, close streams when they are no longer needed.

---

## Frontend (Vue 3) Conventions

Applies to `frontend/src/**/*.vue`.

### Components & Scripts

- Use **Vue 3** with `<script setup lang="ts">`.
- Prefer `const` and explicit TypeScript types to express intent clearly.
- Keep component logic small and focused. Extract complex logic into dedicated **composables/hooks** grouped by responsibility.

### Templates & Structure

- Keep templates as simple and readable as possible.
- If a template grows large or complex, extract parts into smaller reusable components.

### Data Flow

- Follow Vue's recommended one-way data flow: pass data down via **props**, emit events upward rather than mutating parent state directly.

---

## How Copilot Should Behave in This Repo

- Respect all rules above when proposing or editing code.
- Use the appropriate i18n helper (`t()` or `$t()`) instead of hardcoding any user-visible text.
- Prefer minimal, focused diffs and reuse existing utilities, services, composables, and stores when possible.
- Keep all code and comments in English, regardless of the conversation language.
- When making changes to files originally from MCSManager, add a comment at the top of the changed file noting that it has been modified from the original, as required by the Apache 2.0 licence.
