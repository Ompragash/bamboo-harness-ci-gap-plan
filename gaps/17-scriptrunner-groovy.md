# ScriptRunner Groovy automation

## Customer need

The customer runs Groovy automation through ScriptRunner. Portable scripts need a Groovy/JDK environment, arguments, variables, secrets, exit handling, and declared outputs. Scripts that call Bamboo services or APIs require a behavior rewrite rather than only a runtime replacement.

## How Bamboo handles it

ScriptRunner executes Groovy inside or alongside Bamboo. Generic scripts use the available Groovy/JDK runtime and task variables. Bamboo-specific scripts can import Bamboo classes, use injected objects, and change plans or other server state.

```text
Bamboo/agent provides Groovy and Bamboo bindings
-> ScriptRunner supplies script, arguments, and variables
-> script executes or changes Bamboo state
```

ScriptRunner for Bamboo was retired in December 2022, and support ended in May 2023.

## Harness implementation

Recommendation: execute portable Groovy scripts through a native Run step using the explicit Harness Windows Java build image. Do not build a Groovy plugin.

The same `harness/windows-java-build:<jdk-tag>` family used for Maven and Ant contains one supported Groovy release. The pipeline references the required JDK tag directly and runs the repository script.

```text
Harness Run step
-> explicit harness/windows-java-build:temurin17 image
-> groovy .\ci\task.groovy <arguments>
-> exit status and declared Harness outputs
```

A reusable Run Step Template standardizes script path, arguments, environment, secrets, output variables, and failure behavior. A Groovy plugin would only wrap the same process invocation and would not recreate the valuable Bamboo-specific APIs.

Portable scripts run directly. Scripts that use Bamboo plans, repositories, deployments, server services, or injected variables are rewritten using Harness outputs, templates, pipeline chaining/triggers, connectors, or reviewed Harness APIs. Only the Groovy/JDK pairs selected for the POC are packaged; no runtime installation occurs during execution.

## What we still need to confirm

- Which scripts are hard POC blockers?
- Which Groovy/JDK combinations are required?
- Which scripts import Bamboo classes or use injected Bamboo APIs?
- Which outputs or external side effects must be preserved?

## Customer position

- Portable Groovy uses a native Run step with an explicit Harness-owned Java build image.
- No Groovy plugin is required.
- Bamboo-coupled scripts are rewritten around Harness capabilities, not emulated.
- Supported Groovy/JDK pairs are bounded by the POC inventory.

## Sources

- [ScriptRunner for Bamboo retirement](https://www.scriptrunnerhq.com/atlassian-apps/bamboo/scriptrunner-for-bamboo)
- [Apache Groovy command-line tool](https://groovy-lang.org/groovy.html)
- [Harness Run step](https://developer.harness.io/docs/continuous-integration/use-ci/run-step-settings/)
- [Harness Run Step Templates](https://developer.harness.io/docs/platform/templates/run-step-template-quickstart/)
