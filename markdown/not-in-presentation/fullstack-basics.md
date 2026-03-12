# Section 1: Full Stack Basics

Content for slides and speech covering the architecture, why it matters, and how the skill maps to it.

---

## Slide 1: The Polaris Full-Stack Sandwich

**Slide content:**

```
Client Request
     |
  GraphQL + RBAC        (Scala api-server)
     |
  Go gRPC Handler       (business logic)
     |
  DAO Layer              (data access interface)
     |
  PostgreSQL             (per-customer DB)
```

> Every feature you ship touches this stack top to bottom. Six layers, three languages, dozens of files.

**Speaker notes:**

So what does "full stack" actually mean at Rubrik? It's not React and Node. A client hits our GraphQL API in the Scala api-server, which does RBAC checks and routes down to a Go gRPC handler. That handler talks to a DAO layer backed by SQLBoiler, which hits a per-customer PostgreSQL database. Six layers, three languages, and every new feature needs to wire through all of them.

---

## Slide 2: What Each Layer Actually Does

**Slide content (brief bullets, maybe a table):**

| Layer | Language | Key job | Files you'll touch |
|-------|----------|---------|-------------------|
| DB Migration | Go (DSL) | Schema changes — `CreateTable`, `AlterTable` | `migrator/schema/m0XXX-*.go` |
| SQLBoiler Models | Generated | Typed Go structs from DB tables | `db-models/customer/` |
| DAO | Go | Clean interface over SQLBoiler — CRUD + mocks | `internal/<component>dao/` |
| Proto + gRPC | Protobuf | Service contract — RPCs, messages, field masks | `<service>/proto/<service>.proto` |
| Go Handler | Go | Business logic, validation, conversion | `service/svc/`, `service/validator/` |
| GraphQL + RBAC | Scala | External API — schema, resolvers, auth | `api-server/`, `global-api/` |

**Speaker notes:**

At the bottom, database migrations are written in Go using a migrator DSL — no raw SQL. SQLBoiler generates typed models from your tables. The DAO layer wraps that with a clean interface so your handlers never talk to the DB directly. Then your proto file defines the contract — RPCs, messages, field masks. The Go handler implements the actual logic. And at the top, the Scala api-server exposes everything as GraphQL with RBAC. Each layer depends on the one below it. You build bottom-up.

---

## Slide 3: The Problem — This Is a Lot of Boilerplate

**Slide content:**

- A new RPC touches **20-30+ files** across 3 languages
- You need to know: migrator DSL, SQLBoiler, proto conventions, Go handler patterns, Scala GraphQL, RBAC policies
- Even experienced engineers spend hours on the wiring
- The patterns are repetitive — but the details are brutal

> "It's not hard. It's just... a lot."

**Speaker notes:**

None of this is conceptually difficult. It's all well-defined patterns. But doing it from scratch means touching 20-plus files, remembering three different codegen commands, matching proto field names to Go struct tags to GraphQL types. It's tedious. And if you're new to a layer, you're reverse-engineering existing PRs to figure out the conventions. Which is where we started asking: what if Claude just knew the conventions?

---

> Skill architecture diagram and details moved to `understanding-the-fullstack-skill.md`.

---

## Performance Principles (Quick Hit)

> Consider weaving these into the architecture slides or making a brief callout slide.

- **Field masks** — only return what the client asks for
- **Batch, don't loop** — DataLoader / deferred resolvers, not N+1
- **Index your ORDER BY** — LIMIT without an index still scans everything
- **Cache hot paths** — Redis for shared state, LRU for in-process
- **Safe migrations** — additive only, zero downtime

**Speaker notes:**

Worth mentioning performance briefly since it comes up in every full-stack review. Our patterns bake in some defaults: field masks so you don't fetch data you don't need, the DAO layer encouraging batched queries over loops, indexed sort columns, additive-only migrations for zero-downtime deployments. These are part of the scaffolding the skill produces, not something you bolt on after.
