# ScriptRunner Groovy

| Field | Value |
| --- | --- |
| Bamboo plugin key | groovyrunner:scriptrunner.generic |
| Provider | Third party, ScriptRunner |
| Customer version(s) | Not provided |
| Harness CSV status | No |
| Scope | CI, Windows image and reusable template |
| Recommended Harness approach | Execute the existing Groovy file on a maintained Groovy/JDK image through a native Run template |
| Solution type | C. Language or tool image |
| Discovery required | Yes |
| Planning confidence | Medium |

## 1. What this Bamboo task does

ScriptRunner executes Groovy code inside or alongside Bamboo so teams can automate custom behavior not covered by standard tasks. Depending on the script, it may be a simple build utility or may call Bamboo internals and administration APIs.

The product was retired for Bamboo effective December 2022 and support ended in May 2023, which makes migration of active scripts important.

## 2. How it works in Bamboo

Bamboo job → ScriptRunner Groovy task → Groovy runtime with script and bindings → filesystem, tools, APIs, or external systems → console/output and task result.

Material behavior is in each script. A script that only manipulates files differs from one using Bamboo Java APIs, Grape dependencies, credentials, or administrative state.

## 3. How the customer uses it

Confirmed customer usage: the generic ScriptRunner task is present. No scripts, Groovy/JDK versions, bindings, dependencies, Bamboo API calls, or outputs were supplied.

Typical plugin capability: run arbitrary Groovy for custom CI logic.

Customer usage context: not confirmed from the available source material.

Smallest question: Provide the active Groovy scripts and identify any Bamboo-specific APIs, injected bindings, Grape dependencies, or private repositories they use.

## 4. What Harness supports today

Harness Run steps manage image, command, environment, secrets, outputs, reports, timeouts, and failure strategies. Apache Groovy runs directly on Windows with a JDK using the groovy executable.

The reviewed ci-images repository has no Windows Groovy image. The CSV says No because a maintained Windows Groovy/JDK runtime and migration template are missing, not because Groovy needs a task plugin.

## 5. The actual gap

For portable scripts, the gap is a pinned Windows Groovy/JDK image and documented input/output convention. For scripts importing Bamboo classes or mutating Bamboo administration state, the gap is a functional rewrite against Harness APIs or pipeline constructs, which cannot be sized without reading the script.

## 6. Recommended Harness solution

Recommendation: run portable scripts directly on a maintained Groovy/JDK image using a reusable Run step template, and separately refactor Bamboo-coupled scripts after code review.

The customer selects image, script path, arguments, working directory, secret inputs, and named output file. Harness provides versioning, secrets, logs, outputs, timeout, failure strategy, RBAC, and audit.

Engineering packages pinned Groovy/JDK, adds smoke tests and an output convention, and documents Grape/private repository settings. We should not build a thin groovy-task plugin that only invokes the same runtime.

Result: portable Groovy continues to run with a governed native experience, while genuine Bamboo API dependencies are visible migration items.

## 7. Proposed implementation shape

- Base: shared Windows Java image family when JDK and size permit; distinct Groovy tag/variant.
- Contents: pinned JDK and Groovy; no customer dependencies or scripts.
- Project-owned: .groovy files, libraries, Grape coordinates, and domain behavior.
- Template inputs: script, arguments, workdir, environment/secrets, repository settings, output file, timeout, failure strategy.
- Outputs: explicit Harness output file or variables; never scrape arbitrary logs.
- Qualification: portable script, dependency resolution, proxy/CA, private repository, spaces, exit codes, cancellation, and secret masking.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which Groovy/JDK versions are required? | Defines image tags and compatibility. | Customer |
| Do scripts import Bamboo APIs or use injected Bamboo objects? | Determines direct execution versus rewrite. | Customer |
| Are Grape or private dependencies used? | Requires repository, CA/proxy, and credentials. | Customer |
| What values must later steps consume? | Defines a stable output contract. | Customer |

## 9. Validation plan

Classify every active script as portable, Bamboo-coupled, or administrative. Run one portable representative on Windows Kubernetes with exact arguments, dependencies, proxy/CA, paths with spaces, output capture, non-zero failure, cancellation, and secret masking. For coupled scripts, produce a behavior map and prove the replacement with a non-production target before migration.

## 10. Dependencies and risks

- Blocking: active scripts are unavailable.
- Planning: generic task name reveals nothing about script behavior.
- Implementation: Bamboo API coupling, Grape/private dependencies, and output conventions.
- Long-term maintenance: retired source plugin and Groovy/JDK version lifecycle.

## 11. Planning estimate

1 to 2 engineering weeks is the required planning bucket, including qualification contingency. The development portion is included in one shared engineering week for Maven/Ant/Node/Groovy and is not additive by row. This assumes one actual Groovy/JDK pairing and reusable registry/build automation. Bamboo-coupled script rewrites and production lifecycle are not included.

## 12. What we can tell the customer now

- Harness supports governed Windows Run execution for portable Groovy when a compatible Groovy/JDK image is supplied; Harness does not currently provide the proposed maintained image.
- A maintained Windows Groovy/JDK image gives a repeatable platform experience without a redundant plugin.
- ScriptRunner for Bamboo is retired, so active scripts should be inventoried.
- Any Bamboo API coupling must be identified before confirming migration effort.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 31.

### Bamboo/vendor

- [ScriptRunner for Bamboo retirement](https://www.scriptrunnerhq.com/atlassian-apps/bamboo/scriptrunner-for-bamboo)
- [Apache Groovy installation](https://groovy-lang.org/install.html)
- [Buildkite command step](https://buildkite.com/docs/pipelines/configure/step-types/command-step)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/development-guides/ci-windows.md
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/platform/templates/run-step-template-quickstart.md
- [harness-community/ci-images at 9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc](https://github.com/harness-community/ci-images/tree/9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc)
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/RunStepInfo.java

Confidence: Medium.
