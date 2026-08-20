# Artifact handoff

## Customer need

The customer uses Bamboo's Artifact Download task to place producer artifacts into a consumer workspace. The active producer, artifact definition, branch/build selector, destination, and retention semantics are not known.

The migration must preserve artifact identity and reproducibility, not only copy bytes.

## What Bamboo provides

Bamboo associates artifact definitions with producer plans/jobs and stores shared artifacts. A consumer task selects a source plan/build and artifact name, resolves the retained artifact, downloads it to a destination, and fails when required artifacts are unavailable.

```text
producer plan/job
-> Bamboo artifact definition and retained build artifact
-> consumer build selector
-> destination in consumer workspace
```

## Harness today

Steps in one CI stage share a workspace. For separate stages or pipelines, Harness can publish to JFrog or another approved repository and pass the exact coordinate/version as an output or runtime input. Connector-backed steps, plugins, or a governed template retrieve it. CI cache is for reusable cache data, not an immutable release-artifact system of record.

## Gap

The missing item is a migration contract for Bamboo's producer/build selection: exact version, digest, repository path, branch policy, retention, and rerun behavior. “Latest successful” can be reproduced, but silently resolving a different artifact on rerun weakens provenance.

## Recommended approach

Recommendation: use the shared workspace inside one CI stage and an immutable artifact repository contract across stages or pipelines.

Map Bamboo source plan/build selection to an explicit producer pipeline output. The consumer receives the repository, path/coordinate, immutable version, and digest. Use a connector-backed native/plugin step when supported, or one governed download template for the selected repository. A new generic artifact plugin is not required before the repository contract is known.

## POC experience

Demonstrate both applicable patterns:

```text
same CI stage: producer step -> shared workspace path -> consumer step

separate pipelines: publish artifact + digest
-> chain/trigger with version input
-> connector-backed download
-> verify digest
```

Proposed template inputs, not final Harness YAML: connector, repository, immutable path/version, destination, expected digest, retry, and timeout.

## Productized direction

Publish a versioned artifact-handoff template and runbook for the customer's approved repositories. If a later product enhancement is considered, it should preserve immutable producer identity and provenance rather than imitate an opaque latest-build selector.

## Discovery required

- Which source plans, branches, artifact names, and build selectors are active?
- Are transfers within a plan/stage or across independent pipelines?
- Which artifact repository, retention, checksum, and provenance policy is approved?

## Validation

Verify one same-stage and one cross-pipeline transfer. Test exact version/digest, unavailable artifact failure, paths with spaces, retry, permissions, secret masking, retention, consumer rerun, and traceability to the producer execution.

## Effort and ownership

- POC and reusable contract: less than 1 engineering week for one repository and representative flow.
- Likely ownership: CI + HAR for repository integration.

## What we can tell the customer

- Harness already supports shared workspace and repository-backed artifact handoff.
- The migration will make cross-pipeline artifact identity explicit and reproducible.
- We need one active Bamboo selector to preserve its branch and build behavior.

## Sources

- [Atlassian sharing artifacts](https://confluence.atlassian.com/bamboo/sharing-artifacts-359400060.html)
- [Harness share data across steps and stages](https://developer.harness.io/docs/continuous-integration/use-ci/caching-ci-data/share-ci-data-across-steps-and-stages/)
- [Harness upload artifacts to JFrog](https://developer.harness.io/docs/continuous-integration/use-ci/build-and-upload-artifacts/upload-artifacts/upload-artifacts-to-jfrog/)
