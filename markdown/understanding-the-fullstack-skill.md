# Understanding the Full-Stack Skill

The `add-grpc-gql-api` skill is a single orchestrator that walks through the entire stack. It routes based on what you're trying to do, picks the right template, and delegates to focused sub-skills for the DAO and handler layers. Each sub-skill has its own reference patterns pulled from the actual codebase.

---

## Diagram

```mermaid
graph TD
    User["User: 'Add a new RPC to list cloud accounts'"]
    User --> S1

    S1["Step 1: Gather inputs + plan"]
    S1 --> Route{"Operation type?"}
    Route -->|Mutation| TM["TEMPLATE-MUTATION.md"]
    Route -->|Single query| TQS["TEMPLATE-QUERY-SINGLE.md"]
    Route -->|Connection query| TQC["TEMPLATE-QUERY-CONNECTION.md"]
    Route -->|Add field| TAF["TEMPLATE-ADD-FIELD.md"]

    TM & TQS & TQC --> S2["Step 2: Define RPC in proto"]
    S2 --> S3["Step 3: Regenerate proto artifacts"]

    S3 --> S4["Step 4: Scaffold DAO layer"]
    S4 -->|"invokes"| dao_skill
    dao_skill --> S5

    subgraph dao_skill ["add-dao-crud sub-skill"]
        DAO_SKILL["SKILL.md"]
        DAO_PAT["dao-patterns.md"]
        DAO_TEST["dao-test-patterns.md"]
    end

    S5["Step 5: Add Go handler"]
    S5 -->|"invokes"| rpc_skill
    rpc_skill --> S6

    subgraph rpc_skill ["add-rpc-handler sub-skill"]
        RPC_SKILL["SKILL.md"]
        RPC_DAO["handler-dao.md"]
        RPC_DIRECT["handler-direct-db.md"]
        RPC_CONV["handler-proto-conversion.md"]
        RPC_FM["handler-fieldmask.md"]
        RPC_ERR["handler-errors-and-db.md"]
        RPC_TEST_D["handler-tests-direct-db.md"]
        RPC_TEST_M["handler-tests-dao.md"]
    end

    S6["Step 6: Wire Scala + generate GraphQL schema"]
    S6 --> S7["Step 7: Deploy to dev instance"]

    style User fill:#4a90d9,stroke:#2c5f8a,color:#fff
    style Route fill:#e8744f,stroke:#b5522d,color:#fff
    style dao_skill fill:#2d2d2d,stroke:#f5a623,color:#fff
    style rpc_skill fill:#2d2d2d,stroke:#50b87a,color:#fff
```

## Slide bullets

- **One entry point**: `add-grpc-gql-api` — the orchestrator
- **Plans with you first**: gathers inputs, asks clarifying questions, drafts a plan
- **Invokes sub-skills as needed**: each sub-skill is a focused specialist
- **Modular**: not every feature needs every sub-skill (e.g., skip DB migration if the table already exists)

## Speech

We built a Claude Code skill called `add-grpc-gql-api`. It's an orchestrator — one entry point that knows the whole stack. You describe what you want to build, it plans the work with you, then delegates to modular sub-skills: `add-dao-crud` for the data access layer, `add-rpc-handler` for the Go handler logic.

You interact with one skill. You don't need to know which sub-skills exist or invoke them manually — the orchestrator figures out what's needed. New table required? It scaffolds the DAO. Table already exists? Skips that step. Want to add a field to an existing proto instead of a whole new RPC? There's a separate workflow for that, and the orchestrator routes you there based on what you said.

Each sub-skill has its own reference patterns, templates, and conventions pulled from the actual codebase. They're not generic. They know Rubrik's patterns specifically.
