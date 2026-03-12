# Useful Claude Code Skills for Full-Stack Work

Skills are invoked in Claude Code with `/skill-name`. There are 147 skills in sdmain — these are the ones most relevant to full-stack development.

---

## Full-Stack Feature Skills

### `/add-grpc-gql-api`

The end-to-end skill. Generates everything from proto definition through Go handler to Scala GraphQL schema, RBAC policies, and resolvers. Use this when you're adding a completely new API endpoint.

It orchestrates four sub-skills that can also be used independently when you only need one layer:

- **`/add-db-migration`** — Generates a migration file using the `common-go/migrator` DSL. Handles `CreateTable`, `AlterTable`, column types, indexes, and foreign keys. Assigns the next available migration sequence number.
- **`/add-dao-crud`** — Generates the full DAO layer: interface (`*_dao.go`), SQLBoiler-backed implementation (`*_dao_impl.go`), mock (`*_dao_mock.go`), and tests (`*_dao_test.go`). Accepts `boil.ContextExecutor` so callers can pass transactions.
- **`/add-rpc-handler`** — Generates a Go gRPC handler that implements the proto-generated interface. Includes request validation, DAO calls, proto conversion, and proper gRPC error returns. Wires into the existing service struct.
- **`/add-field-to-existing-proto`** — Adds a new field to an existing proto message with correct field numbering, runs codegen, and updates the handler and DAO layers to use the new field. Handles the full vertical slice.

---

## Other Useful Skills

### `/deployment`

Builds and deploys your services to a dev environment. Smarter than a raw build — it identifies which services actually need rebuilding based on your changes, so you're not waiting 10+ minutes for an API server build you don't need.

### `/bug-hunt`

Investigates bugs by analyzing code paths, recent changes, and related test failures. Give it a bug description or JIRA ticket and it traces through the relevant layers to identify likely root causes. Especially useful when a bug spans multiple services.

### `/code-walk`

Fast code navigation using Universal Ctags. Indexes Go, C++, and Python symbols across the codebase. Use it to find definitions, list struct members, trace inheritance, and find callers — roughly 4x faster than grep-based lookups.
