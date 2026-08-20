# Maven dependencies processor

| Field | Value |
| --- | --- |
| Bamboo plugin key | plugins.maven:task.mvn.dependencies.processor |
| Provider | Bamboo native |
| Customer version(s) | Bundled with the customer Maven plugin versions |
| Harness CSV status | No |
| Scope | CI pipeline orchestration |
| Recommended Harness approach | Model active dependencies with ordered stages, pipeline chaining, triggers, and immutable artifact inputs |
| Solution type | D. Reusable Harness orchestration template |
| Discovery required | Yes |
| Planning confidence | Medium |

## 1. What this Bamboo task does

This processor scans Maven project metadata and uses SNAPSHOT relationships to update Bamboo plan dependencies. Bamboo can then order dependent builds and apply dependency-blocking rules.

It is not a Maven compiler or package resolver. Its distinctive behavior is changing Bamboo's build graph from POM metadata.

## 2. How it works in Bamboo

Bamboo build → dependency processor scans the POM → matches Maven coordinates to Bamboo plans → updates automatic plan dependencies → Bamboo schedules upstream and downstream builds according to plan rules.

Atlassian documents automatic dependency updates on each run and special parent-child linking for SNAPSHOT dependencies. The output is orchestration metadata inside Bamboo, not a Windows executable result.

## 3. How the customer uses it

Confirmed customer usage: the task is present and follows the same Maven-version estate as the Maven builder row.

Typical plugin capability: automatically maintain plan dependency relationships derived from Maven POMs.

Customer usage context: not confirmed from the available source material.

Smallest question: Do the exported plans rely on automatically created producer-consumer plan links, dependency blocking, or only ordinary Maven dependency resolution?

## 4. What Harness supports today

Harness can express known producer-consumer relationships with ordered stages, pipeline chaining, triggers, and immutable artifact version inputs. Those constructs can represent the required build flow when the graph is known.

Harness Build Intelligence already provides dependency-aware build acceleration for supported Maven, Gradle, and Bazel workloads. Current documentation limits it to Linux, so it is not the Windows orchestration path for this POC. On Windows, ordered stages, pipeline chaining, triggers, and immutable artifact inputs provide the producer-consumer workflow.

The CSV says No because Harness does not reproduce Bamboo's automatic POM-to-plan graph mutation. The migration solves the active producer-consumer outcome with Harness-native orchestration.

## 5. The actual gap

Harness can model builds in the correct order, but the current active relationships must be made explicit during migration. A Windows plugin is not required because the target capability is pipeline orchestration rather than a build-tool operation.

## 6. Recommended Harness solution

Recommendation: map the customer's active producer-consumer relationships to Harness pipeline chaining, triggers, ordered stages, and immutable artifact inputs.

The user experience should be explicit dependency inputs and observable upstream pipeline status. Engineering initially provides a migration runbook and reusable orchestration pattern.

We should not build a Windows plugin for this orchestration function. The migration runbook should record the Bamboo-derived relationships once and create governed Harness templates that make producer, consumer, trigger, and artifact-version inputs visible.

Result: known build relationships can be demonstrated in the POC without pretending that Build Intelligence supplies Bamboo plan-dependency behavior.

## 7. Proposed implementation shape

- Immediate pattern: producer publishes an immutable artifact coordinate and version.
- Consumer: receives the version through a trigger or pipeline input and downloads the exact artifact.
- Orchestration: ordered stages for one pipeline; pipeline chaining or triggers for separate pipelines.
- Governance: connector access, RBAC, audit, retry, failure strategy, and visible upstream execution links.
- Migration output: a reviewed mapping from Bamboo plan/POM relationships to Harness pipeline identifiers and artifact coordinates.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which plans and POM coordinates are linked automatically today? | Establishes whether the behavior is used at all and its scale. | Customer |
| Are dependency blocking and transitive scheduling required? | Determines whether static chaining is sufficient. | Customer |
| Which producer/consumer trigger and blocking behavior must the template preserve? | Defines the explicit Harness orchestration inputs. | Customer |
| How are artifacts versioned and promoted between the plans? | Defines the immutable handoff contract. | Customer |

## 9. Validation plan

Select one real producer-consumer chain. Prove that a producer execution publishes an immutable version, triggers or gates the consumer, passes the exact version, and surfaces failures without consuming an older artifact. Compare execution ordering and rerun behavior with the Bamboo plan and record the mapping used.

## 10. Dependencies and risks

- Blocking: the inventory does not confirm that automatic plan linking is used.
- Planning: an incomplete plan/POM export can omit active relationships.
- Implementation: cyclic dependencies, authorization, and coordinate-to-pipeline ownership.
- Long-term maintenance: the explicit relationship and artifact contract needs an owner when pipelines change.

## 11. Planning estimate

<1 engineering week for a reusable orchestration pattern and one representative producer-consumer mapping after the active relationships are supplied.

## 12. What we can tell the customer now

- Harness can model producer-consumer CI flows with ordered stages, pipeline chaining, triggers, and immutable artifact inputs.
- Build Intelligence covers dependency-aware build acceleration where supported; the Windows producer-consumer flow uses pipeline chaining, triggers, ordered stages, and immutable artifact inputs.
- We need one exported dependency chain to configure and validate the POC mapping.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 4.

### Bamboo/vendor

- [Atlassian: Setting up plan build dependencies](https://confluence.atlassian.com/bamboo0902/setting-up-plan-build-dependencies-1236931619.html)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/build-and-upload-artifacts/build-intelligence.md
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/caching-ci-data/share-ci-data-across-steps-and-stages.md
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/RunStepInfo.java

Confidence: Medium.
