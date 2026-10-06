# Repository guidance

These instructions apply to the entire repository. Keep this file focused on shared project conventions; personal tooling, credentials, and author preferences belong in local configuration.

## Project overview

Intellitrex is a cryptocurrency research and analytics prototype. It combines market charts, technical indicators, externally generated forecasts, an AI consultant, and portfolio analysis.

The application uses Next.js 16.2.2 (App Router), React 19, strict TypeScript, and Tailwind CSS 4. Model Settings persists provider configurations in SQLite using Node's built-in `node:sqlite` driver. Use Node.js `>=24.15.0 <25` and npm with the committed `package-lock.json`.

Read `README.md` for current functionality and setup. Consult `docs/model-settings-setup.md` for operational procedures and `docs/database-architecture.md` before changing persistence. Treat implementation plans as context; verify behavior against current code.

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## Repository layout

- `src/app/`: App Router pages, layouts, global styles, and API route handlers.
- `src/components/`: Shared UI; `src/components/admin/` contains Model Settings.
- `src/lib/`: Shared utilities, constants, and browser authentication context.
- `src/lib/server/`: Server configuration, operator authentication, credential encryption, HTTP helpers, and provider client.
- `src/lib/server/model-configurations/`: Configuration types, validation, business rules, repository interface, and service composition.
- `src/lib/server/database/sqlite/`: SQLite connection, repository adapter, and migrations.
- `scripts/model-storage.ts`: Setup, migration, verification, backup, and legacy-key import commands.
- `tests/`: Feature checks with temporary storage and mocked providers.
- `docs/`: Architecture, implementation, and operational documentation.

## Development commands

Run commands from the repository root.

| Command | Purpose |
| --- | --- |
| `npm ci` | Install locked dependencies. |
| `npm run dev` | Start the development server. |
| `npm run build` | Build the production application. |
| `npm start` | Serve an existing production build. |
| `npm run lint` | Run ESLint. |
| `npx tsc --noEmit` | Check TypeScript types. |
| `npm test` | Run automated feature checks. |
| `npm run model-settings:setup` | Initialize local operator credentials and storage interactively. |
| `npm run db:migrate` | Apply database migrations. |
| `npm run db:verify` | Verify database schema, integrity, and credentials. |
| `npm run db:backup -- /absolute/path/new.sqlite` | Create a consistent database backup. |
| `npm run model-settings:import` | Import an existing environment OpenAI key as an inactive configuration. |

Setup and storage commands can modify local configuration or data. Run them only when needed for the task, against the intended environment. Normal tests should use isolated temporary databases rather than the developer's database.

## Implementation conventions

- Follow nearby code conventions and keep changes focused. Avoid unrelated refactors or dependency upgrades.
- Use the `@/*` alias for imports rooted in `src/`. Preserve strict typing and validate untrusted input at server boundaries.
- Keep server modules server-only. Add client boundaries where browser APIs, hooks, or interaction require them; do not import database or secret-handling modules into client code.
- Use existing components, theme styles, and the `cn` utility for class composition. Preserve light/dark appearance, accessible labels, keyboard behavior, and clear loading and error states when changing UI.
- Routes handle HTTP and access control; configuration services handle business rules; the repository adapter owns SQL and storage transactions.
- Use existing HTTP helpers and sanitized errors for Model Settings endpoints. Preserve operator checks and same-origin checks on protected mutations.
- Keep validated model infrastructure configuration in `src/lib/server/config.ts`. Prefer passing configuration through service composition over scattered environment reads.
- Keep provider requests in the shared provider client. Preserve bounded input, timeouts, and safe error handling.

## Model configuration and persistence rules

- Settings responses must exclude both plaintext provider keys and encrypted credential envelopes.
- New configurations are inactive. At most one configuration may be active; activation must switch entries atomically.
- Reject deletion of an active configuration. Changing its endpoint, model, or key deactivates it; a name-only change preserves active status.
- Keep these invariants enforced within storage transactions, including under concurrent requests. Use parameterized SQL.
- Chat uses the explicitly active saved configuration. Do not introduce silent fallback to another entry or `OPENAI_API_KEY`.
- Keep SQL and `node:sqlite` details behind the repository interface. Add migrations for schema changes rather than editing released migrations.
- Production migrations are explicit operations. Avoid database writes at module import or during builds.
- Preserve existing encryption keys. A replacement key cannot decrypt saved credentials without a deliberate re-encryption procedure.
- The current deployment requires persistent local storage on one host. Do not assume ephemeral serverless storage or independent writable replicas support this design.

## Security and data handling

Never commit or log real credentials, operator hashes, signing or encryption secrets, database files, backups, or decrypted records. Keep local secrets in untracked `.env.local`; `.env.example` should contain documentation and placeholders only. Never expose secrets through `NEXT_PUBLIC_` variables or browser storage.

Use mocked provider calls for automated tests. Live connection tests can incur charges and should be performed only within the requested scope. Preserve consistent SQLite backup handling rather than copying only an open main database file without its WAL.

## Known product boundaries

User sign-in, subscriptions, and chat history currently use browser-local prototype behavior; they are separate from server-protected Model Settings operator authentication. Do not treat them as verified accounts or server-enforced quotas.

Only model configurations are persisted in the application database. Forecast training runs outside this repository. Market data, sentiment, forecasts, and portfolio results depend on external services. Do not describe planned trading, billing, or learning features as implemented without verifying the code.

## Validation and documentation

For behavioral changes, add or update meaningful tests covering the changed behavior, especially authentication, credential handling, storage invariants, and provider failures. The test entry point is `tests/run.ts`; register new test files there when needed.

Run relevant tests, TypeScript checks, and lint for code changes. Run a production build for changes affecting application integration or framework behavior. Documentation-only changes generally need a diff and accuracy review rather than an application build.

The README records existing effect-related lint failures and build-time Google Fonts requests. Verify current failures before attributing them to your change; report remaining failures without suppressing unrelated rules.

Update relevant documentation when changing configuration, commands, API behavior, storage, or deployment requirements. Summarize what changed, what was verified, and any remaining limitations. Do not commit or push unless the user requests it.
