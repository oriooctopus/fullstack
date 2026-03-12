# Source PRs

PRs that define the `add-grpc-gql-api` skill and its sub-skills. These are the foundation of the presentation.

---

## The Master Skill (Orchestrator)

| PR | Title | Author | What it does |
|----|-------|--------|-------------|
| [#155175](https://github.com/scaledata/sdmain/pull/155175) | Add add-grpc-gql-api Claude Code skill | Jackson763 | The original orchestrator skill. Step-by-step workflow covering proto definition, codegen, RBAC wiring, and schema generation. Includes `SKILL.md` and `TEMPLATES.md` with worked examples for mutations, single queries, and connection queries. |

## Sub-Skills (Modular Components)

| PR | Title | Author | What it adds |
|----|-------|--------|-------------|
| [#161211](https://github.com/scaledata/sdmain/pull/161211) | Add DAO CRUD skill and integrate into add-grpc-gql-api workflow | chetkoziol | `add-dao-crud` sub-skill — scaffolds DAO interface + SQLBoiler implementation for database CRUD. Integrated into master skill as optional Steps 4-5. |
| [#161212](https://github.com/scaledata/sdmain/pull/161212) | Add DB migration skill and integrate into add-grpc-gql-api workflow | chetkoziol | `add-db-migration` sub-skill — scaffolds migration files using the common-go/migrator DSL. Integrated as optional Step 4 when new tables/schema changes are needed. |
| [#161213](https://github.com/scaledata/sdmain/pull/161213) | Add RPC handler skill and integrate into add-grpc-gql-api workflow | chetkoziol | `add-rpc-handler` sub-skill — scaffolds Go gRPC handlers (DAO-backed or direct-DB). Integrated as an alternative to mock-only implementations. |

## Extended Workflows

| PR | Title | Author | What it adds |
|----|-------|--------|-------------|
| [#161184](https://github.com/scaledata/sdmain/pull/161184) | Add field-to-existing-proto workflow to add-grpc-gql-api skill | oriooctopus | Second workflow for adding a field to an existing proto message. Routes between "Add new RPC" and "Add field to existing proto" based on user intent. Includes worked example with ToolStats + active_alerts_count. |
