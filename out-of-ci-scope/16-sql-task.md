# SQL task scope decision

## Decision

The Appfire SQL task is outside the active CI capability plan. Its documented contract executes inline SQL or a script file against a configured JDBC database profile and can write raw, line-preserving, wiki, or HTML output. That is database change or database-operation behavior, not a CI product gap merely because a build agent can issue the command.

Harness DB DevOps is designed for governed, version-controlled database schema changes in deployment pipelines. CD also owns database deployment orchestration where DB DevOps is not licensed or applicable. Those are the correct primary workstreams for a task that changes a shared or deployed database.

## Conditional CI case

This capability can re-enter CI planning only if the customer shows that an active use is limited to an ephemeral test database, such as:

- creating test fixtures before integration tests;
- validating generated SQL against an isolated database;
- querying a build-local database to assert test outcomes;
- tearing down a database created exclusively for the current CI execution.

That bounded case would use an engine-specific client image or service, Harness secrets, and a governed Run template. It would not justify a generic multi-database plugin until actual engine, authentication, transaction, output, and cleanup needs show repeated integration logic.

## Ownership and effort

- Primary owner: DB DevOps or CD.
- CI owner only for a proven ephemeral test-fixture case.
- No CI POC engineering estimate is carried in this pack.

## Sources

- [Appfire SQL task fields and examples](https://appfire.atlassian.net/wiki/spaces/SUPPORT/pages/89139613/_ExamplesForAddTaskAction)
- [Harness Database DevOps overview](https://developer.harness.io/docs/database-devops/overview/)
- [Harness Database DevOps FAQs](https://developer.harness.io/docs/faqs/db-devops-faqs/)
