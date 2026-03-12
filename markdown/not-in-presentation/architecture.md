# Polaris Backend Architecture

The layers a full-stack feature touches, from bottom to top.

---

## 1. Database (PostgreSQL + Migrator DSL)

Every Polaris microservice owns its schema in a per-customer PostgreSQL database. Schema changes are written in Go using the `common-go/migrator` DSL — no raw SQL.

**Key operations:**
- `CreateTable` — new table with columns, indexes, foreign keys
- `AlterTable` + `AddNotNullColumn` — add a column with a default value
- `AlterTable` + `AddNullableColumn` — add an optional column

**File pattern:**
```
polaris/src/rubrik/<service>/migrator/schema/m0XXX-<description>.go
```

Migrations run automatically at service startup in order. Each migration file has a unique sequence number. The migrator handles idempotency and rollback.

---

## 2. SQLBoiler Models (Generated ORM Layer)

After a migration lands, SQLBoiler generates Go structs that map 1:1 to database tables. These are the typed representations used by the DAO layer.

**File pattern:**
```
polaris/src/rubrik/db-models/customer/<table_name>.go
```

SQLBoiler models are **generated, not hand-written**. You run `sqlboiler` after your migration and commit the output.

---

## 3. DAO (Data Access Object)

The DAO layer wraps SQLBoiler with a clean interface. Every DAO follows the same structure:

| File | Purpose |
|------|---------|
| `*_dao.go` | Interface definition (e.g., `List`, `Create`, `Update`, `Delete`) |
| `*_dao_impl.go` | SQLBoiler-backed implementation |
| `*_dao_mock.go` | Generated mock for unit tests |
| `*_dao_test.go` | Tests using `go-sqlmock` |

**File pattern:**
```
polaris/src/rubrik/<service>/internal/<component>dao/
```

The interface allows the handler layer to be tested without a real database.

---

## 4. Proto + gRPC (Service Contract)

Protobuf files define the API contract: RPC methods, request/response messages, and field types. A single `.proto` file generates:

- **Go server/client stubs** — used by the handler
- **Python client bindings** — used by the SDK and tests
- **Validation rules** — via proto annotations

**File pattern:**
```
polaris/src/rubrik/<service>/proto/<service>.proto
```

After editing a proto, you run codegen to regenerate the Go and Python files. The generated code lives alongside the proto.

**Key concepts:**
- `FieldMask` — clients specify which fields to return/update
- `google.protobuf.Timestamp` — standard timestamp type
- Enums — defined in proto, used everywhere downstream

---

## 5. Go gRPC Handler (Business Logic)

The handler implements the gRPC interface generated from the proto. This is where business logic lives.

**Typical handler structure:**

```
polaris/src/rubrik/<service>/service/
├── svc/                    # Handler implementations
│   └── <rpc>_handler.go
├── validator/rpc/          # Request validation
│   └── <rpc>_validator.go
└── converter/              # Proto ↔ domain model conversion
    └── <type>_converter.go
```

**What a handler does:**
1. Validates the request (via validator)
2. Calls DAO methods to read/write data
3. Converts between proto messages and domain models
4. Returns the response or a gRPC status error

Handlers are unit-tested with mocked DAOs.

---

## 6. Scala api-server (GraphQL + RBAC)

The `api-server` is the Scala gateway that exposes gRPC services to the outside world via GraphQL. Adding a new API here involves several pieces:

### GraphQL Schema
```
polaris/src/rubrik/api-server/documentation/schema/schema-internal.graphql
```
Defines the GraphQL types, queries, and mutations that map to the underlying gRPC RPCs.

### RBAC Policies
```
polaris/src/rubrik/api-server/app/services/<service>/rbac/
```
Role-based access control: which roles can call which operations. Policies are defined in Scala and reference authorization tags.

### Resolvers
```
polaris/src/rubrik/api-server/app/apps/<service>/schema/
```
Scala resolvers that bridge GraphQL queries/mutations to gRPC calls. They handle argument parsing, call the gRPC stub, and format the response.

### DAL (Data Abstraction Layer)
The api-server has its own DAL that wraps gRPC client calls, providing a clean interface for resolvers.

### Global API Registration
```
polaris/src/rubrik/global-api/schema/schema.go
```
New RPCs must be registered in the global API schema so they're routable.

---

## 7. Python SDK (Generated Client Bindings)

Python bindings are auto-generated from protos and provide the client-side interface used by integration tests and the Rubrik Python SDK.

**File pattern:**
```
polaris/src/rubrik/sdk_internal/grpc/
```

---

## How It All Fits Together

```
Client Request
     │
     ▼
┌─────────────┐
│  GraphQL     │  Scala api-server
│  + RBAC      │  (authorization, schema, resolvers)
└──────┬──────┘
       │ gRPC call
       ▼
┌─────────────┐
│  Go Handler  │  Business logic, validation, conversion
└──────┬──────┘
       │ DAO interface
       ▼
┌─────────────┐
│  DAO Layer   │  SQLBoiler implementation
└──────┬──────┘
       │ SQL queries
       ▼
┌─────────────┐
│  PostgreSQL  │  Per-customer database
└─────────────┘
```

**The standard order for building a new feature:**
1. DB migration (create/alter tables)
2. SQLBoiler model generation
3. DAO (interface + implementation + tests)
4. Proto definition (RPC + messages)
5. Go handler (business logic + validation + tests)
6. Scala api-server (GraphQL schema + RBAC + resolvers)

Each layer depends on the one below it. PRs can land independently as long as they follow this order.
