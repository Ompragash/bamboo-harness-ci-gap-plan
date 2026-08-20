# ScriptRunner Groovy automation

## Customer need

The customer has a generic ScriptRunner Groovy task. Without the scripts, it is unknown whether this is a portable build utility or code that calls Bamboo services, plan objects, variables, or administration APIs.

Portable scripts need Groovy, a compatible JDK, environment variables, and exit-code handling. Bamboo-coupled scripts need behavioral redesign, not only a Groovy runtime.

## What Bamboo provides

ScriptRunner can execute Groovy inside or alongside Bamboo and may expose Bamboo application bindings and APIs. The product was retired in December 2022 and support ended in May 2023. Public implementation source for the exact generic task was not found, so script exports are the authoritative parity evidence.

```text
ScriptRunner task
-> Groovy runtime plus possible Bamboo bindings
-> customer script
-> process result or Bamboo-side mutation
```

## Harness today

Harness can run a Groovy file in a Windows Run step when the image supplies a compatible JDK and Groovy. Templates provide structured inputs, secrets, environment, logs, outputs, timeout, failure strategy, RBAC, and audit. Harness APIs, pipeline chaining/triggers, and outputs replace Bamboo-specific management or orchestration behaviors where needed.

## Gap

For portable scripts, the only gap is a trusted runtime and reusable invocation. A dedicated `groovy-task` plugin adds little. For scripts that import Bamboo classes or mutate Bamboo state, no generic plugin can make them portable; each behavior must map to a Harness pipeline primitive or API.

## Recommended approach

Recommendation: execute portable Groovy directly through a governed template and reuse the shared Java resolver only when runtime selection is required.

Do not create a Groovy plugin for simple script execution. Use a customer image in the POC. If multiple portable versions later justify product support, add Groovy archives as a bounded ToolSpec on top of the shared Java provider. Rewrite Bamboo-bound logic using Harness outputs, templates, chaining/triggers, or a reviewed Harness API client.

## POC experience

Proposed template inputs, not final Harness YAML:

```yaml
java:
  distribution: temurin
  version: "17"
groovy:
  version: "4.0.28"
scriptFile: ci/versioning.groovy
arguments: [--mode, verify]
outputFile: .harness/outputs.env
```

The example versions are illustrative. The POC selects one portable script, uses a customer-provided runtime image, and maps declared outputs to later Harness steps.

## Productized direction

Keep the template as the product solution when scripts are portable. Add shared resolver support only for approved Groovy/JDK pairs and sources. Maintain Bamboo-to-Harness rewrite patterns for variables, plan dependency behavior, repository operations, and APIs. Do not attempt to emulate Bamboo Java services inside a plugin.

## Discovery required

- Provide the active Groovy scripts and identify Bamboo imports, bindings, filesystem assumptions, network calls, and required outputs.
- Which Groovy/JDK pairs and Windows constraints block the POC?
- Which scripts mutate Bamboo plans, variables, repositories, or deployments?

## Validation

Classify each selected script as portable or Bamboo-bound. For one portable sample, verify runtime identity, arguments, environment/secrets, file paths, outputs, exit-code failure, network/proxy/CA, cancellation, and secret masking. For a coupled sample, compare the redesigned Harness outcome rather than byte-for-byte script execution.

## Effort and ownership

- Portable POC template: included in the 1 to 2 week shared toolchain workstream.
- Coupled scripts: discovery required and estimated per behavior.
- Likely ownership: CI for template; customer/application team for script semantics; Platform/API owner for approved replacements.

## What we can tell the customer

- Portable Groovy scripts can run directly in a governed Harness step with a compatible JDK/Groovy runtime.
- A Groovy plugin is unnecessary unless repeated runtime provisioning later justifies a shared contract.
- Scripts coupled to Bamboo APIs will be mapped to Harness behavior rather than emulated.
- The retired ScriptRunner dependency makes script export a priority.

## Sources

- [ScriptRunner for Bamboo retirement](https://www.scriptrunnerhq.com/atlassian-apps/bamboo/scriptrunner-for-bamboo)
- [Apache Groovy installation](https://groovy-lang.org/install.html)
- [Harness Windows CI](https://developer.harness.io/docs/continuous-integration/development-guides/ci-windows/)
- [Harness Run step templates](https://developer.harness.io/docs/platform/templates/run-step-template-quickstart/)
