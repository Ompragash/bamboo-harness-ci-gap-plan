# Maven dependency orchestration

## Customer need

The customer has Bamboo's Maven Dependencies Processor in its inventory. The important need is likely to start or block downstream builds when producer Maven artifacts or SNAPSHOT relationships change. The active plan relationships and required selection behavior are not yet known.

This is orchestration metadata, not a Windows Maven execution feature.

## What Bamboo provides

Bamboo can inspect Maven project metadata and update plan dependencies from Maven relationships. Bamboo's dependency model can then order builds, block dependents, and propagate build activity through the plan graph.

```text
POM relationship
-> Bamboo plan dependency
-> producer build
-> dependent plan scheduling
```

The useful outcome is a reliable producer-consumer graph. Automatic POM discovery is Bamboo-specific convenience, and the customer may not need it for every plan.

## Harness today

Harness can represent the outcome explicitly with ordered stages, pipeline chaining, completion triggers, runtime inputs, and immutable artifact coordinates. Build Intelligence can optimize supported Maven build work but does not replace cross-pipeline orchestration.

## Gap

Harness does not automatically translate the customer's Bamboo-discovered plan graph. The migration must identify active producer-consumer relationships and express them as explicit Harness orchestration and artifact contracts.

## Recommended approach

Recommendation: model the active relationships with Harness pipeline chaining/triggers, ordered stages, and immutable artifact version inputs.

A plugin is not required. An execution container cannot safely mutate the management-plane graph, and implicit POM discovery can hide ownership and version-selection rules. Explicit orchestration is easier to audit and reproduce.

## POC experience

Select one active chain and configure:

```text
producer pipeline
-> publish immutable artifact and digest
-> completion trigger or chained pipeline
-> pass artifact version as input
-> consumer downloads exact version
```

The POC should also demonstrate expected behavior when the producer fails, the artifact is absent, or the consumer is rerun.

## Productized direction

Keep the explicit Harness model. Provide a migration runbook or template for repeated graph shapes. Build Intelligence can complement this by identifying affected builds where its supported model applies; pipeline chaining and triggers remain the general orchestration mechanisms.

## Discovery required

- Which processor-generated dependencies are active POC blockers?
- Is the trigger source a successful producer build, a POM change, or an artifact version?
- Are latest-success selection, branch matching, blocking, or fan-out/fan-in semantics used?

## Validation

Compare one producer-consumer chain with Bamboo. Verify success, failure, rerun, branch behavior, artifact identity/digest, duplicate event handling, and auditability of the selected producer execution.

## Effort and ownership

- POC and product template: less than 1 engineering week for one representative chain.
- Likely ownership: CI; artifact repository ownership may involve HAR.

## What we can tell the customer

- Harness can model the dependency outcome through native chaining, triggers, stages, and artifact inputs.
- No Windows plugin or POM-scanning container is required for the POC.
- We need one active generated relationship to preserve its exact scheduling behavior.

## Sources

- [Atlassian plan build dependencies](https://confluence.atlassian.com/bamboo0902/setting-up-plan-build-dependencies-1236931619.html)
- Harness local evidence: `developer-hub` `1c7c98f1d76bb7b8330d6ffba96f984878a32748`, Build Intelligence and data-sharing docs.
