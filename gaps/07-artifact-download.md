# Artifact handoff

## Customer need

The customer downloads artifacts produced by another Bamboo job or plan into a consumer workspace. Harness must preserve the producer, branch/build selection, artifact name, destination, and failure behavior.

The important requirement is knowing exactly which producer artifact the consumer used.

## How Bamboo handles it

Bamboo stores artifacts against producer plans, jobs, and build results. The Artifact Download task selects a producer/build and artifact definition, then downloads the retained files into the current agent workspace.

```text
Bamboo producer build
-> retained named artifact
-> consumer selects producer/build/artifact
-> artifact downloaded to consumer workspace
```

## Harness implementation

Recommendation: use the shared Harness workspace inside one CI stage and an immutable artifact repository contract across stages or pipelines.

For a single stage, no download component is needed because steps share the stage workspace. For separate pipelines, the producer publishes to JFrog or another Harness-supported repository and returns the immutable artifact version and digest. Pipeline chaining or a trigger passes those values to the consumer, which downloads and verifies the exact artifact.

```text
Harness producer
-> publish artifact version + digest
-> pass values through pipeline chaining/trigger
-> consumer downloads exact version
-> digest verified before use
```

Harness should provide one reusable artifact-handoff template for each approved repository. If JFrog is selected, it uses the repaired Harness Artifactory plugin. Cache Intelligence remains dependency/build caching and is not used as the artifact system of record.

## What we still need to confirm

- Which producer/build selectors and branch rules are active?
- Are artifacts transferred within one plan or between independent plans?
- Which artifact repository, retention, and checksum policy should Harness use?

## Customer position

- Harness already provides the required workspace and pipeline orchestration primitives.
- Cross-pipeline artifacts will be immutable, verified, and traceable to the producer.
- No new generic Windows artifact plugin is planned unless the selected repository lacks a supported integration.

## Sources

- [Atlassian sharing artifacts](https://confluence.atlassian.com/bamboo/sharing-artifacts-359400060.html)
- [Harness share data across steps and stages](https://developer.harness.io/docs/continuous-integration/use-ci/caching-ci-data/share-ci-data-across-steps-and-stages/)
- [Harness upload artifacts to JFrog](https://developer.harness.io/docs/continuous-integration/use-ci/build-and-upload-artifacts/upload-artifacts/upload-artifacts-to-jfrog/)
