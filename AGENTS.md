# Instructions for the agent

Read this file at the start of every session. Keep it short and correct; update it when the project changes.

## Project
Study Groups helps TU-Sofia students find, join and organize study groups and sessions. See [SPEC.md](SPEC.md).

## Commands
- Install: No application dependencies at this stage. Run the checks with Node.js 22 from the repository root.
- Run the lecture MCP server: `npm run --silent mcp`.
- Check the Exercise 2 specification: `node checks/ex2.mjs`.
- Test the lecture MCP server: `node mcp-server/check.mjs -- node mcp-server/server.js`.
- Run `npm test` to check the Exercise 2 specification and the Exercise 1 MCP server. Application startup, application tests and coverage are pending.

## Conventions
- Planned folders are `web/` for React/Vite, `src/http/` for Express, `src/services/` for business rules, `src/data/` for Prisma access and `prisma/` for schema and migrations.
- Validate transport input at the boundary. Services return domain errors, and HTTP handlers map them to status codes and readable messages.

## Layers
Requests arrive two ways and both end in the same place:

- HTTP route handler → service → data access → database
- MCP tool → **the same service** → data access → database

A route handler reads the request and turns a result into a status code. A service holds the
rules and knows nothing about HTTP or about MCP. Only the data layer writes SQL. An MCP tool
that talks to the database directly is a second, unguarded way into your data: never add one.

## The model feature
- Keep the provider integration in one adapter in `src/model/`, called by the recommendation service. Configure it with `OPENAI_MODEL` and `OPENAI_API_KEY`.
- The key comes from an environment variable and is never committed or logged.
- Every call has a timeout and a deterministic fallback, so the app still works when the provider is down or rate-limited.
- The model's output is re-checked against real data before anything is saved or shown.
- Tests never call the provider: they pass in a fake client.

## The MCP server
- Preserve the Exercise 1 lecture tools in `mcp-server/`.
- The planned project tools call the same services as HTTP. They resolve the user via `MCP_TOKEN` and enforce the same authorization rules.
- Adding a tool means adding a row to the table in `SPEC.md` too.

## Tests and coverage
- Give each criterion in `SPEC.md` a unique AC id. When implementing it, add a test whose title contains that id.
- The coverage gate runs in CI: 100% of the service layer, 80% overall.
- A test is evidence. Changing a test so the code passes destroys the evidence.

## Always ask before
- deleting files or folders
- editing a migration that has already run
- installing or removing dependencies
- pushing, tagging or changing CI
- anything that touches secrets or `.env`
- changing an authentication or authorization rule
- adding, renaming or removing an MCP tool
- changing a coverage threshold
- sending user data to a third-party model

## Never
- commit secrets, passwords, API keys or an OAuth client secret
- weaken or delete a test to make it pass
- lower a coverage threshold or switch off a CI job to make the build green
- add an MCP tool that reaches past the service layer
- log a prompt that contains somebody's personal data
