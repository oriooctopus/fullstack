# Common Mistakes by Layer

Real patterns pulled from reviewer comments on landed PRs in `scaledata/sdmain`.

---

## 1. DB Migrations

### Nullable columns when NOT NULL + default is correct

The most common flag. Authors default to `AddNullableColumn` even for boolean flags that should be `AddNotNullColumn` with `Default(false)`. The Edith bot catches this every time.

> **PR #68104:** Used `AddNullableColumn("is_paused", dtypes.Boolean())` — reviewer required `AddNotNullColumn` with `Default(false)`.

### Missing query documentation and EXPLAIN analysis

The Alfred bot requires every migration PR to list expected queries, their frequency, and `EXPLAIN` output proving indexes are used. Authors never proactively include this — it's flagged on every single migration PR.

### Migration number collisions

Migration sequence numbers are assigned at PR creation time. If another migration lands first, your number conflicts and you have to renumber mid-PR.

> **PR #57566:** File renamed from `m0186-*` to `m0187-*` after a collision with a concurrent migration.

### Single-value enums

Adding an enum with one value looks like a code smell. Reviewers will ask "do we plan on adding more?" Enums are expensive to alter on large MySQL tables.

> **PR #57566:** `Enum([]string{"SERVERS_AND_APPS"})` — reviewer flagged it as odd.

### Stale migration files in the diff

Not rebasing before opening a PR causes previously-merged migration files to appear in your diff, confusing reviewers.

> **PR #57582:** Two migration files from a prior PR appeared in the diff. Reviewer: "This was covered in earlier PR, right?"

### Inconsistent timestamp handling

`created_at` is NOT NULL but `updated_at` is nullable with no default — creating rows where `updated_at` is NULL until the first update. Be consistent.

---

## 2. DAO Layer

### SQLBoiler `.Upsert()` with multi-column unique keys

SQLBoiler's `Upsert` doesn't work correctly when the unique key spans multiple columns. You must manually get-then-insert/update inside a transaction.

> **PR #60844:** Author self-noted the limitation. **PR #63475:** Reviewer explicitly flagged it with a link to the internal wiki.

### Passing `*sql.DB` instead of `boil.ContextExecutor`

Using `*sql.DB` in the DAO interface prevents callers from passing a transaction. Accept `boil.ContextExecutor` so callers can pass either `*sql.DB` or `*sql.Tx`.

> **PR #60844:** Reviewer requested `boil.ContextExecutor`. Author acknowledged with a TODO.

### Overly broad UPDATE/archive operations

`UpdateAll` without a sufficiently narrow WHERE clause can modify rows that shouldn't be touched (e.g., re-archiving already-archived rows).

> **PR #70115:** Initial implementation updated all rows including already-archived ones. Fix: add `archived_at IS NULL` to the WHERE clause.

### Silently swallowing `sql.ErrNoRows`

Returning `nil` when no rows are found hides invalid inputs from callers and makes debugging hard.

> **PR #63475:** Reviewer: "Would we not want to know that the request was invalid in this case?"

### `err == sql.ErrNoRows` instead of `errors.Is()`

Direct error comparison breaks when errors are wrapped. Always use `errors.Is(err, sql.ErrNoRows)`.

> **PR #87853:** Both DAO methods used direct comparison instead of `errors.Is`.

### Unnecessary FK pre-validation queries

DAOs shouldn't query other tables to validate foreign keys exist before writing — let DB constraints handle it.

> **PR #70115:** Reviewer: "Why do we need to validate the cloud account state inside this method, which is just to get from the mapping table?"

### Cross-component direct DB table access

A DAO in one component importing and querying another component's DB models creates tight coupling.

> **PR #70115:** After merge, reviewer noted: "We shouldn't be reading from a `cloudaccounts` table directly from `exocompute`."

### Too many primitive parameters

Interface methods with 8+ positional string parameters are error-prone. Use a struct.

> **PR #87853:** `CreateOrUpdateDevopsCustomerOrganizationsInTx` had 8 positional string args.

### TODOs without JIRA tickets

Every TODO must include the author and a JIRA reference: `TODO(username) [SPARK-XXXXXX]: description`.

> **PR #87853, #63475:** Alfred bot flagged missing JIRA ticket on TODOs.

---

## 3. Proto Definitions

### Backward-incompatible field changes

Reusing a proto field number with a different type is a wire-format-breaking change. When removing a field, mark it `reserved`.

> **PR #76716:** Changed field 1 from `string account_id` to `context.RequestContext req_ctx`. Backward compat bot: "Field type change is not allowed."

### Enum missing `_UNSPECIFIED = 0`

All new proto enums must have a `FOO_UNSPECIFIED = 0` sentinel as the first value. Without it, unset fields are ambiguous.

> **PR #75878:** `CloudAccountStatus` started with `CONNECTING = 0` instead of `CLOUD_ACCOUNT_STATUS_UNSPECIFIED = 0`.

### `bool success` in reply messages

gRPC signals success/failure through the error return, not a boolean field. A `success` field is redundant and misleading.

> **PR #64252, #4334:** Both had `bool success = 1` in their reply messages. Callers check `err`, not `success`.

### Reply echoing request fields

Don't return fields the caller already knows (e.g., `aws_account_id` in the reply when it was in the request).

> **PR #4334:** Reviewer: "I guess the aws_account_id will always be the same as the one in the Request." Author removed it.

### Consuming new fields/RPCs in the same release

Services upgrade independently. Client code consuming a new field/RPC must not land in the same release as the server code that provides it.

> **PR #2809:** Reviewer: "You have added a new proto field and expecting it to be set at client side in same release? This can cause backward incompatibility."

### GQL annotations in the same PR as the proto change

Adding `option (api.graphql_method)` annotations alongside the new RPC causes Sail codegen failures. Must be a follow-up PR.

> **PR #72497:** Reviewer: "Remove all the GQL annotations in this PR, and do a stacked PR with these changes."

---

## 4. Go gRPC Handlers

### Inconsistent RPC naming

RPC names, request/reply message names, and field names must all follow `{CloudProvider}{Verb}{Noun}` and be derived from the same base.

> **PR #32488:** `GetIsAzureFeatureEnabled` should have been `AzureIsFeatureEnabled` to match the cloud-provider-prefix convention.

### Wrong validator instantiation

Creating a `NewValidatorImpl()` inline instead of using the pre-injected `service.rpcValidator` field.

> **PR #32488:** Reviewer: "You don't need to create Validator here, it's part of the service attribute: `service.rpcValidator`."

### No UNIMPLEMENTED fallback at new RPC call sites

Any client calling a newly added RPC must handle `codes.Unimplemented` gracefully during the rolling upgrade window.

> **PR #72497:** Edith: "Include fallback logic to handle UNIMPLEMENTED gRPC code if this client code is being introduced in the same release."

### Logging sensitive credentials

Access keys, secret keys, and tokens must never be logged, even at DEBUG level. Use `secrets.String` wrappers.

> **PR #4334:** Reviewer: "Please don't log access key" and "Why aren't we using `secrets.String`?"

### Proto type used directly as cache/storage representation

Storing proto types directly in cache/GCS means any proto field change corrupts serialized data. Use an intermediate struct.

> **PR #72497:** Reviewer: "We probably want to decouple the protobuf and json representation for compatibility purposes."

### Insufficient test coverage

Every handler needs tests for: happy path, DAO error, validation failure, and nil/empty returns. Coverage below 50% on new lines blocks landing.

> **PR #72497:** Coverage dropped to 0% on initial submission. Multiple revisions needed to reach threshold.

---

## 5. Scala GraphQL + RBAC

### Wrong RBAC policy wired to a field

Copy-pasting a `NewField` block and forgetting to update the `rbacPolicyOpt` to point to the correct policy.

> **PR #12266:** `rbacPolicyOpt = Some(rbacPolicies.setIpWhitelistEnabled)` was wired to the wrong operation.

### Missing RBAC unit tests

Every new RBAC policy needs tests for: correct access, denial without resource, and denial with wrong operation.

> **PR #12266, #33910:** Reviewers flagged missing RBAC policy tests.

### Verbs in query field names

Query fields are nouns (`ociCloudAccounts`), not verbs (`getOciCloudAccounts`). Mutations keep verbs.

> **PR #75878:** Reviewer: "Don't use verbs in names of query fields. Rename to `ociCloudAccounts`."

### Always-true `Boolean!` instead of `Void`

When a mutation returns nothing meaningful, use `Void` not `Boolean!` that's always true.

> **PR #76716:** Reviewer: "Looking at the implementation, 'success' will always be true unless an error is thrown. Would it make sense to use `Void`?"

### Proto type names leaked into GraphQL schema

Types named `SaasAppWorkloadSizeEstimatorReply` in GraphQL — never expose `*Reply`, `*Req`, `*Response` suffixes.

> **PR #33910:** Reviewer required rename to `SalesforceObjectsSizeEstimate`.

### Non-batching resolver (missing `NamedFetcher`)

Calling a downstream gRPC service directly in a `resolve` lambda causes N+1 queries. Use `NamedFetcher` + `DeferredValue`.

> **PR #63306, #33910, #75878:** Edith flagged: "Ensure that a non-batching performance anti-pattern is not being introduced."

### Using `NewField` instead of api-server v2

`NewField` is the legacy DSL. New endpoints must use the api-server v2 system.

> **PR #63306:** Alfred: "Adding new fields using `NewField` is not allowed. Please add this API using api-server v2."

### UUIDs typed as `String` instead of `UUID` scalar

ID fields that hold UUIDs must use the `UUID` custom scalar type, not `String`.

> **PR #33910, #75878:** `workloadId: String!` and `cloudAccountId: String!` flagged — should be `UUID`.

### Missing `UnifiedFeatureFlag` / `Hidden` tag

Non-GA fields must include `Hidden :: UnifiedFeatureFlag(YOUR_FLAG) :: Nil` in tags.

> **PR #33910:** Reviewer: "Do we need to add UFF flag here?"

### Not registering new types in `global-api/schema.go`

Every new GraphQL type (Connection, Edge, standalone) in `schema-internal.graphql` needs a corresponding Go struct in `schema.go`.

> **PR #75878, #136626:** All new types required corresponding Go struct registration.

### Wrong/missing GQL field descriptions

All fields need descriptions in present tense, from the client's perspective, without referencing internal proto names.

> **PR #76716:** Reviewer: "Please fix the comment — `OciCloudAccountEnable` is not a term that appears in GraphQL schema."
