# SQL task

| Field | Value |
| --- | --- |
| Bamboo plugin key | com.appfire.bamboo.sql:sql |
| Provider | Appfire |
| Customer version(s) | Not provided |
| Harness CSV status | No |
| Scope | CI, database discovery and tool image |
| Recommended Harness approach | Discover engine and result contract; prefer a database-specific tool image/template before any generic plugin |
| Solution type | I. Discovery required before selecting implementation |
| Discovery required | Yes |
| Planning confidence | Low |

## 1. What this Bamboo task does

Appfire's SQL task executes inline SQL or a script file against a configured JDBC database profile. It centralizes connection settings and can write results in raw, line-preserving, wiki, or HTML formats.

The task may be used for schema setup, test fixtures, validation queries, or database changes. Those uses have different transaction and safety requirements.

## 2. How it works in Bamboo

Bamboo job → Appfire SQL task → selected JDBC profile/URL, user, password, and driver → inline/file SQL → database → task status and optional result file.

Material inputs include task key, inline/file mode, JDBC URL/profile, authentication, script, and output format/path. Driver and database semantics are external dependencies.

## 3. How the customer uses it

Confirmed customer usage: the task is in the inventory. No database engine, driver, authentication, scripts, transaction mode, output use, or migration/DACPAC behavior is supplied. The CSV notes that Harness DB DevOps is not licensed.

Typical plugin capability: execute SQL through JDBC and optionally persist query output.

Customer usage context: not confirmed from the available source material.

Smallest question: Which database engines and authentication modes are used, and are these validation/setup scripts or schema deployments?

## 4. What Harness supports today

Harness Run steps can execute a database vendor CLI in a maintained Windows image while managing secrets, environment, logs, outputs, timeout, and failure strategy. For SQL Server, sqlcmd or SqlPackage is a focused path. A generic JDBC path would require a JRE plus licensed/redistributable drivers and a defined structured-output contract.

No verified universal database task exists in the reviewed CI implementation. The CSV says No because the needed engine and semantic contract are unknown, not because every SQL invocation needs a plugin.

## 5. The actual gap

The gap may be only a qualified database client image/template, or it may include multi-engine JDBC profiles, output conversion, transaction/delimiter behavior, and Windows integrated authentication. These are materially different implementations.

Schema deployment through DACPAC or migration tooling should not be conflated with running test setup or validation SQL.

## 6. Recommended Harness solution

Recommendation: discover the active database contract, then use a database-specific Windows client image and reusable Run template when one engine covers the need.

The customer configures engine-specific connection inputs, secret references, inline or repository script path, transaction/failure behavior, and output artifact/variables. Harness supplies governance, logs, secrets, outputs, timeout, failure strategy, and template versioning.

Engineering packages the approved client and tests auth, CA/proxy, exit codes, encoding, and result capture. Consider an Appfire-like JDBC plugin only if multiple engines and structured cross-database outputs are confirmed. We should not bundle unlicensed drivers or create a generic database plugin before discovery.

Result: the smallest safe database execution capability is demonstrated without committing to unsupported engine parity.

## 7. Proposed implementation shape

- SQL Server path: Windows image with pinned sqlcmd and SqlPackage only when DACPAC is confirmed.
- Other single-engine path: vendor-supported CLI with required CA/auth libraries.
- Template inputs: host/URL, database, auth secret, script/inline SQL, variables, transaction/fail mode, output path/format, timeout.
- Outputs: exit status and explicit non-sensitive values; full query data as an artifact where appropriate.
- Conditional plugin: JDBC engine/profile, approved drivers, named outputs, formats, retry/transient-error model, and deterministic redaction.
- Safety: non-production POC database, least privilege, and no logging of credentials or sensitive result rows.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which database engines and driver versions are active? | Selects CLI/image versus JDBC plugin. | Customer |
| Which authentication modes, including Windows integrated auth, are used? | Determines native libraries and domain testing. | Customer |
| Inline or file scripts, delimiters, transactions, and output formats? | Defines exact task semantics. | Customer |
| Are DACPACs or migration frameworks involved? | This is a different workflow from query execution. | Customer |
| Can required drivers be redistributed? | May block a packaged generic image. | Vendor / Legal |

## 9. Validation plan

Run representative non-production scripts on the target Windows Kubernetes node. Verify authentication, private CA/proxy, inline and file mode, variables, transactions, delimiters, Unicode, paths with spaces, expected result format, non-zero/error handling, transient failure behavior, timeout, secret masking, and least-privilege access. Confirm no sensitive output is exposed.

## 10. Dependencies and risks

- Blocking: database engines, scripts, test endpoint, and credentials are unknown.
- Planning: validation SQL, migrations, and DACPAC deployment are separate use cases.
- Implementation: driver licensing, integrated auth, transactions, and sensitive outputs.
- Long-term maintenance: vendor clients and database-driver security updates.

## 11. Planning estimate

Discovery required before estimate. A single SQL Server CLI image/template may fit within 1 to 2 engineering weeks after the engine and auth contract are known, but it is not included in the shared Maven/Ant/Node/Groovy image estimate. A multi-engine JDBC integration must be estimated after engines, drivers, and result semantics are known.

## 12. What we can tell the customer now

- Harness can provide governed database-tool execution with native secrets, logs, outputs, and failure controls.
- A SQL Server-only need likely maps to a maintained client image and reusable template.
- We will not commit to a universal SQL plugin until engines and output behavior are known.
- Database deployment and CI test/setup SQL need to be separated in discovery.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 30.

### Bamboo/vendor

- [Appfire: SQL task examples and fields](https://appfire.atlassian.net/wiki/spaces/SUPPORT/pages/89139613/_ExamplesForAddTaskAction)
- [Appfire SQL for Bamboo version history](https://marketplace.atlassian.com/apps/1215265/sql-for-bamboo/version-history)
- [Microsoft Azure SQL action](https://github.com/Azure/sql-action)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/development-guides/ci-windows.md
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/RunStepInfo.java

Confidence: Low.
