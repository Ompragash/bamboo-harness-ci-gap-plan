# Artifact Download

| Field | Value |
| --- | --- |
| Bamboo plugin key | bamboo-artifact-downloader-plugin:artifactdownloadertask |
| Provider | Bamboo native |
| Customer version(s) | Not provided |
| Harness CSV status | Partial |
| Scope | CI/CD artifact handoff; CI portion analyzed here |
| Recommended Harness approach | Native workspace within a stage and immutable repository handoff across stages or pipelines |
| Solution type | B. Existing native capability plus qualification |
| Discovery required | Yes |
| Planning confidence | Medium |

## 1. What this Bamboo task does

The task copies artifacts produced by another Bamboo plan, job, or build into the current job workspace. Bamboo resolves a producer/build selection, knows the artifact definition, downloads it, and optionally places it at a chosen destination.

Its important behavior is artifact identity and producer/build selection, not generic file copying.

## 2. How it works in Bamboo

Producer plan/job → Bamboo artifact definition and retained artifact → consumer Artifact Download task → selected build artifact → consumer workspace.

Material semantics can include source plan, branch, latest successful or specific build, artifact name, destination, and failure when the artifact is unavailable.

## 3. How the customer uses it

Confirmed customer usage: the inventory includes the task, but provides no producer, selection, repository, or retention details.

Typical plugin capability: move a build artifact between Bamboo jobs or plans without manually configuring an external repository command.

Customer usage context: not confirmed from the available source material.

Smallest question: Does each use copy within one plan, select a previous/latest producer build, or retrieve an immutable version from an artifact repository?

## 4. What Harness supports today

Steps in one Harness CI stage share the stage workspace. For cross-stage or cross-pipeline handoff, Harness guidance is to publish artifacts to an external repository and retrieve the exact artifact later. Harness has JFrog upload capability and repository connectors; Run steps can retrieve from supported repositories. Save/Restore Cache is for cache data and should not be the release-artifact system of record.

The CSV is Partial because the core handoff is supported, but Bamboo's plan/build/artifact selection model does not map one-for-one without knowing the customer's selection semantics.

## 5. The actual gap

The gap is a documented artifact contract for cross-stage/pipeline cases: producer identity, immutable version, path, checksum, retention, and consumer input. If the customer depends on “latest successful build of plan X,” migration also needs an explicit selection rule and traceability.

## 6. Recommended Harness solution

Recommendation: use the shared workspace inside one CI stage and an immutable artifact repository contract across stages or pipelines.

The producer publishes an artifact with coordinate/version/checksum metadata. The consumer receives that version as a pipeline input and downloads it using a connector-backed step or governed template. Harness supplies secrets, RBAC, audit, logs, outputs, retry, and failure handling.

Engineering work is qualification and a reusable handoff template/runbook, not a new downloader plugin. We should not use Cache Intelligence as a substitute for an immutable artifact repository.

Result: each consumer can identify and reproduce exactly which producer artifact it used.

## 7. Proposed implementation shape

- Same-stage: shared workspace with explicit relative paths.
- Cross-stage/pipeline: JFrog, S3, or the customer's approved artifact repository.
- Producer outputs: repository, coordinate/path, immutable version, digest, and optional metadata.
- Consumer inputs: same fields plus destination and checksum policy.
- Template: connector, repository, version, source path, destination, checksum, retries, timeout, and outputs.
- Runbook: mapping for latest-success behavior, retention, rerun, and unavailable artifacts.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which producers, branch rules, and build-selection modes are used? | Defines whether immutable inputs replace latest-success lookup. | Customer |
| Which artifact repositories and connectors are approved? | Determines the native/template path. | Customer |
| Are artifacts passed within a plan or across independent plans? | Separates workspace from repository handoff. | Customer |
| Are checksum, retention, and provenance requirements defined? | Sets acceptance and governance criteria. | Customer / Product |

## 9. Validation plan

Demonstrate one same-stage transfer and one real cross-pipeline artifact. Verify exact version and checksum, paths with spaces, unavailable artifact failure, retry behavior, secret masking, reruns, and traceability back to the producer execution. Confirm that consumers never silently select a different artifact.

## 10. Dependencies and risks

- Blocking: no producer/build-selection examples.
- Planning: latest-success semantics can undermine reproducibility if copied blindly.
- Implementation: repository API, proxy/CA, retention, and permissions.
- Long-term maintenance: artifact naming and version contracts need ownership.

## 11. Planning estimate

Qualification only if an approved repository and immutable version contract already exist. <1 engineering week for a reusable template/runbook after mappings are supplied. Repository-specific engineering is outside this estimate.

## 12. What we can tell the customer now

- Harness supports in-stage workspace sharing and repository-backed artifact handoff.
- Cross-pipeline artifacts should use immutable versions and checksums, not cache storage.
- A reusable template can provide a native governed experience.
- We need the Bamboo producer and build-selection mappings before confirming parity.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 11.

### Bamboo/vendor

- [Atlassian: Sharing artifacts](https://confluence.atlassian.com/bamboo/sharing-artifacts-359400060.html)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/caching-ci-data/share-ci-data-across-steps-and-stages.md
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/build-and-upload-artifacts/upload-artifacts/upload-artifacts-to-jfrog.md
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/caching-ci-data/saving-cache.md

Confidence: Medium.
