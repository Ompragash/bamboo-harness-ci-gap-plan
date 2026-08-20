# Ant

## Customer need

The customer needs to run Ant builds on Windows. The active JDK, Ant version, build file, targets, environment, working directory, and JUnit report configuration have not been supplied.

The desired outcome is a repeatable Java toolchain and task configuration, not merely the presence of `ant.exe` or a separate plugin for every target.

## What Bamboo provides

Bamboo's native Ant task selects configured Ant and JDK capabilities and makes them agent requirements. It accepts a build file, targets and arguments, environment or `ANT_OPTS`, working subdirectory, and JUnit result configuration.

```text
Bamboo Ant task
-> agent with Ant and JDK capabilities
-> build file and targets
-> Ant outcome and optional JUnit results
```

Public Bamboo implementation source is not available without a commercial source license; official task documentation and Specs define the verified contract.

## Harness today

Harness can execute Ant in a Windows Run step and ingest JUnit results. The reviewed `ci-images` repository has no Windows lane. `drone-ant` PR 1 is an unmerged goals-only Go wrapper. Its LTSC 2022 Dockerfile installs JDK 8 and Ant through unpinned Chocolatey packages, and its checked-in empty-goals test does not pass. It lacks build-file, JDK, environment, working-directory, and report inputs.

## Gap

The missing experience is trusted Java/Ant selection and a reusable Ant-oriented contract. The PR does not preserve Bamboo's capability selection or report behavior, and duplicating Java provisioning inside an Ant-only plugin would create maintenance debt.

## Recommended approach

Recommendation: use a customer toolchain image and governed Ant template for the POC; productize a thin Ant contract only if repeated customer use justifies it, backed by the same Java resolver as Maven.

| Option | Decision |
| --- | --- |
| Fixed image | Good POC path for one or two Ant/JDK pairs; avoid a permanent cross-product matrix. |
| Broad image | Rejected as the default because unrelated Node/.NET tools increase size and attack surface. |
| Dedicated dynamic plugin | Conditional; useful only if it calls the shared Java resolver and adds structured Ant/report inputs. |
| Hybrid | Preferred product shape if Ant volume warrants a contract. |
| Customer image | Preferred legacy and POC fallback. |

## POC experience

Proposed template inputs, not final Harness YAML:

```yaml
java:
  distribution: temurin
  version: "17"
ant:
  version: "1.10.15"
buildFile: build.xml
targets: [clean, test, package]
reports: [build/test-results/*.xml]
```

The POC can use the selected Ant and JDK already installed in a customer image. Harness controls environment, secrets, logs, timeout, failure strategy, template governance, and reports.

## Productized direction

If more than a template is justified, harden or supersede the existing PR with build file, targets, arguments, workdir, environment, JUnit paths, exact Ant version, and Java ToolSpec inputs. The implementation must call the shared resolver for Java, support approved mirrors/proxy/CA/checksums, and avoid `latest` or unpinned package installation.

Do not commit to a dedicated plugin until exported configurations show repeated structured behavior beyond a simple invocation.

## Discovery required

- Which Ant/JDK pairs and targets block the POC?
- Are custom Ant distributions, tasks, launchers, or `ANT_HOME` layouts used?
- Which build files, working directories, `ANT_OPTS`, and report globs are configured?
- Are runtimes available only through an internal mirror?

## Validation

Run one representative build on the target LTSC node. Verify exact Ant/JDK identity, custom tasks, private dependency access, paths with spaces, environment, success/failure exit codes, JUnit counts, cold/warm cache behavior, cancellation, and secret masking.

## Effort and ownership

- POC: included in the 1 to 2 week shared toolchain workstream.
- Product thin contract: 1 to 2 engineering weeks after the shared resolver exists and use is confirmed.
- Likely ownership: CI; Platform owns resolver and release controls.

## What we can tell the customer

- Harness can run Ant and publish its test results on Windows with a compatible toolchain.
- The POC will qualify the active Ant/JDK pair rather than promise every combination.
- The existing community PR is not suitable as-is.
- Any product plugin will share Java provisioning with Maven instead of duplicating it.

## Sources

- [Atlassian Ant task](https://confluence.atlassian.com/display/BAMBOO/Ant)
- [Bamboo plugin source availability](https://developer.atlassian.com/server/bamboo/bamboo-plugin-guide/)
- [`drone-ant` PR 1 commit `53b582d4abfbfb7ffb45561b3d42b7c9f468f310`](https://github.com/harness-community/drone-ant/commit/53b582d4abfbfb7ffb45561b3d42b7c9f468f310)
- Harness local evidence: `developer-hub` `1c7c98f1d76bb7b8330d6ffba96f984878a32748`, Windows CI and report docs.
