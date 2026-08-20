# Maven dependency orchestration

## Customer need

The customer uses Bamboo's Maven Dependencies Processor to connect producer and consumer builds based on Maven relationships. The migration must preserve build ordering, downstream triggers, failure blocking, and the exact artifact selected by each consumer.

This capability changes the CI build graph. It is not part of Maven compilation and does not require a Windows runtime image.

## How Bamboo handles it

Bamboo reads Maven project metadata and updates dependencies between Bamboo plans. Bamboo's server-side orchestration then schedules downstream plans when the configured producer completes and applies Bamboo's dependency-blocking rules.

```text
Maven project relationship
-> Bamboo plan dependency
-> producer build completes
-> dependent plan starts or remains blocked
```

The processor saves teams from configuring every Bamboo plan link manually, but the useful result is the producer-consumer graph rather than the processor itself.

## Harness implementation

Recommendation: represent the active graph explicitly with Harness pipeline chaining, triggers, ordered stages, and immutable artifact inputs.

For each active dependency, the producer publishes an artifact version and digest. A completion trigger or pipeline chaining starts the consumer and passes the exact version. The consumer downloads that version rather than performing an untracked “latest successful” lookup.

```text
Harness producer pipeline
-> publish artifact version + digest
-> completion trigger / pipeline chaining
-> Harness consumer receives exact version
-> consumer downloads and verifies artifact
```

Harness should provide a reusable orchestration template and a migration mapping for common Bamboo graph shapes. A Windows plugin is not needed because an execution container should not modify the Harness management-plane graph. Build Intelligence can optimize supported Maven builds, while pipeline chaining and triggers remain the general dependency mechanism.

## What we still need to confirm

- Which generated plan dependencies are active POC blockers?
- Are branch matching, transitive triggers, failure blocking, or fan-out/fan-in used?
- Does any consumer depend on “latest successful” rather than an explicit artifact version?

## Customer position

- Harness provides this through native pipeline orchestration, not a Windows plugin.
- Producer-consumer relationships will be explicit and auditable.
- Cross-pipeline artifacts will use immutable versions and digests.

## Sources

- [Atlassian plan build dependencies](https://confluence.atlassian.com/bamboo0902/setting-up-plan-build-dependencies-1236931619.html)
- [Harness pipeline chaining](https://developer.harness.io/docs/platform/pipelines/pipeline-chaining/)
- [Harness share data across steps and stages](https://developer.harness.io/docs/continuous-integration/use-ci/caching-ci-data/share-ci-data-across-steps-and-stages/)
