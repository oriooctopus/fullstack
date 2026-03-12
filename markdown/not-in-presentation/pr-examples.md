# PR Examples by Sub-Skill

Example landed PRs that demonstrate what each skill automates. Use these in the presentation walkthrough section.

---

## Hero Stack: OCI Cloud Account Feature (by rishabh1907, May-Jul 2025)

The best end-to-end example of the full-stack pattern. One author, all standard Polaris patterns, all 5 layers covered across ~9 weeks:

| PR | Date | Layer | What it does |
|----|------|-------|-------------|
| [#57566](https://github.com/scaledata/sdmain/pull/57566) | May 4 | DB Migration | `CreateTable` for `oci_customer_features` via migrator DSL |
| [#57582](https://github.com/scaledata/sdmain/pull/57582) | May 4 | DB Migration + SQLBoiler | `CreateTable` for permission groups + SQLBoiler models |
| [#63475](https://github.com/scaledata/sdmain/pull/63475) | May 30 | DAO | `ocidao/oci_dao_impl.go` — DAO for OCI cloud account operations |
| [#64252](https://github.com/scaledata/sdmain/pull/64252) | Jun 2 | Proto | `cloudaccountsoci.proto` — RPC definitions |
| [#64729](https://github.com/scaledata/sdmain/pull/64729) | Jun 10 | Go gRPC Handler | `oci_svc_impl.go` — handler implementation |
| [#75878](https://github.com/scaledata/sdmain/pull/75878) | Jul 6 | Scala GraphQL + RBAC | `CloudAccountsRbacPolicies.scala`, `schema-internal.graphql` |
| [#76716](https://github.com/scaledata/sdmain/pull/76716) | Jul 6 | Scala GraphQL + RBAC | GQL enable API, `schema.go` (global-api) |

Also notable: [#136626](https://github.com/scaledata/sdmain/pull/136626) (Jan 2026) — single PR touching proto, Go handler, Scala RBAC, GraphQL, and UI (27 files), but no DB layer (returns static data).

---

## 1. Add New gRPC RPC + GraphQL Wiring

Full pattern: proto definition → codegen (Go/Python) → RBAC → GraphQL schema

| PR                                                           | Title                                      | Why it's good                                                                             |
| ------------------------------------------------------------ | ------------------------------------------ | ----------------------------------------------------------------------------------------- |
| [#12266](https://1226github.com/scaledata/sdmain/pull/12266) | Create AddWhitelistEntries() endpoint      | Most complete end-to-end mutation example. 23 files, touches every layer including tests. |
| [#63306](https://github.com/scaledata/sdmain/pull/63306)     | Add GQL to get k8s object config           | Great query example. 32 files, CDM REST + GraphQL wiring.                                 |
| [#33910](https://github.com/scaledata/sdmain/pull/33910)     | Add GQL changes for size estimator library | Clean query example with full RBAC + GQL + Go handler + tests.                            |

---

## 2. Add Field to Existing Proto

Modify an existing proto message with a new field, regenerate code, implement server-side population.

| PR                                                         | Title                                      | Why it's good                                                                                |
| ---------------------------------------------------------- | ------------------------------------------ | -------------------------------------------------------------------------------------------- |
| [#2809](https://github.com/scaledata/sdmain/pull/2809)     | Include Archival Location in snapshot Info | Simplest clean example. 4 files: proto field + codegen + server-side `ToProto()` population. |
| [#144124](https://github.com/scaledata/sdmain/pull/144124) | Add new fields in UMD for AD Policies      | Most comprehensive. 18 files across proto, C++, Scala, Thrift, Swagger, UI, and tests.       |
| [#68104](https://github.com/scaledata/sdmain/pull/68104)   | Add is_paused to clouddirect_policy table  | Simple boolean field addition. Minimal and focused.                                          |

---

## 3. DB Migration (common-go/migrator DSL)

Creating or altering tables using the migrator DSL.

| PR                                                       | Title                                                 | Why it's good                                                    |
| -------------------------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------------------- |
| [#57566](https://github.com/scaledata/sdmain/pull/57566) | Add DB migration for OCI customer features table      | Clean `CreateTable` example with columns, indexes, foreign keys. |
| [#68104](https://github.com/scaledata/sdmain/pull/68104) | Add is_paused to clouddirect_policy table             | Minimal `AlterTable` + `AddNotNullColumn` with default value.    |
| [#46938](https://github.com/scaledata/sdmain/pull/46938) | Add SLA related fields in clouddirect_nas_share table | `AlterTable` adding multiple nullable columns in one migration.  |

---

## 4. DAO CRUD (Interface + SQLBoiler Implementation)

Scaffolding a data access layer with interface, SQLBoiler impl, mocks, and tests.

| PR                                                       | Title                                                | Why it's good                                                                             |
| -------------------------------------------------------- | ---------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| [#60844](https://github.com/scaledata/sdmain/pull/60844) | Implement DAO for exocloud hosting resource capacity | Most complete self-contained example. Interface + 413-line SQLBoiler impl + mock + tests. |
| [#70115](https://github.com/scaledata/sdmain/pull/70115) | Implement DAO for certificate/cloud account mapping  | Smallest/cleanest example. Only 3 files. List, Add, Expire methods with go-sqlmock tests. |
| [#87853](https://github.com/scaledata/sdmain/pull/87853) | Add DAO layer for devops_customer_organization       | Part of a stack showing the repeatable pattern across multiple tables.                    |

---

## 5. RPC Handler (Go gRPC Handler Implementation)

Implementing the actual handler logic: proto conversions, field masks, error handling, DAO calls.

| PR                                                       | Title                                                   | Why it's good                                                               |
| -------------------------------------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------------- |
| [#4334](https://github.com/scaledata/sdmain/pull/4334)   | Add gRPC call for AWS IAM user ops                      | Full stack: proto → validator → service handler → DAO. 18 files with tests. |
| [#32488](https://github.com/scaledata/sdmain/pull/32488) | Implement RPC request for azure account feature enabled | Full stack with handler, DAO, model, validator, mocks, tests. 14 files.     |
| [#72497](https://github.com/scaledata/sdmain/pull/72497) | Add gRPC endpoint ListKnownIssues in ui-egress-service  | Greenfield endpoint with clean layered architecture. 18 files.              |
