# ScriptRunner Groovy automation

## Customer need

The customer runs Groovy automation through ScriptRunner. Some scripts may be portable build utilities; others may import Bamboo classes, use injected Bamboo objects, or change Bamboo plans and variables.

Harness must run portable scripts in Windows containers and clearly identify Bamboo-coupled scripts that require a behavior rewrite.

## How Bamboo handles it

ScriptRunner executes Groovy inside or alongside the Bamboo application. A generic script can use the installed Groovy/JDK runtime, task variables, arguments, and exit status. A Bamboo-specific script can also call Bamboo services and APIs that do not exist outside Bamboo.

```text
Bamboo agent/server with Groovy and Bamboo APIs
-> ScriptRunner supplies script, variables, and bindings
-> script executes
-> process result or Bamboo-side change is returned
```

ScriptRunner for Bamboo was retired in December 2022 and support ended in May 2023.

## Harness implementation

Recommendation: provide a thin Harness Groovy wrapper on the Harness-maintained Windows Java runtime family.

Each supported Java profile includes one supported Groovy runtime layer and the wrapper. The user selects the Java profile, script file, arguments, environment, secrets, and declared outputs. The Harness abstraction selects the internal image.

```text
Harness Groovy Wrapper
-> Harness Windows Java profile with supported Groovy
-> repository Groovy script + arguments
-> exit status and declared Harness outputs
```

The wrapper does not emulate Bamboo APIs. Portable scripts run directly. Scripts that call Bamboo plans, variables, repositories, deployments, or server services are rewritten using Harness outputs, templates, pipeline chaining/triggers, repository connectors, or reviewed Harness APIs.

Harness should support only the Groovy/JDK pairs required for the POC. The Java runtime is already in the image; no JDK or Groovy installation occurs during the pipeline. A separate Groovy runtime family is unnecessary.

## What we still need to confirm

- Which scripts are active POC blockers?
- Which Groovy/JDK versions are required?
- Which scripts import Bamboo classes or use Bamboo-injected objects/APIs?
- Which outputs or external side effects must be preserved?

## Customer position

- Harness will provide a governed Groovy wrapper on its Java runtime family.
- Portable scripts can run directly in Windows Kubernetes.
- Bamboo-coupled scripts will be rewritten around Harness capabilities, not emulated.
- Supported Groovy/JDK pairs will be packaged and maintained by Harness.

## Sources

- [ScriptRunner for Bamboo retirement](https://www.scriptrunnerhq.com/atlassian-apps/bamboo/scriptrunner-for-bamboo)
- [Apache Groovy installation](https://groovy-lang.org/install.html)
- [Harness Run step templates](https://developer.harness.io/docs/platform/templates/run-step-template-quickstart/)
