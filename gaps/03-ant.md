# Ant

| Field | Value |
| --- | --- |
| Bamboo plugin key | plugins.ant:task.builder.ant |
| Provider | Bamboo native |
| Customer version(s) | Not provided |
| Harness CSV status | No |
| Scope | CI, Windows image and reusable template |
| Recommended Harness approach | Maintained Windows Ant/JDK image with a reusable native Run step template |
| Solution type | C. Language or tool image |
| Discovery required | Yes |
| Planning confidence | Medium |

## 1. What this Bamboo task does

The task invokes Apache Ant targets for a Java build. Bamboo wraps the Ant executable with build-file, target, environment, working-directory, JDK capability, and failure configuration.

Ant remains the build engine. The useful product layer is a predictable tool environment and repeatable task configuration.

## 2. How it works in Bamboo

Bamboo job → Ant task → selected Ant/JDK capability → build.xml targets → exit status and generated test reports.

Material inputs include build file, targets, JVM and Ant options, environment, and working directory. A failed target fails the task. Reports are collected separately.

## 3. How the customer uses it

Confirmed customer usage: Ant appears in the inventory, but no Ant or JDK versions are listed.

Typical plugin capability: run build.xml targets such as clean, compile, test, and package with configured properties.

Customer usage context: not confirmed from the available source material.

Smallest question: Which Ant/JDK versions, build file paths, targets, and ANT_OPTS are present in the exported plans?

## 4. What Harness supports today

Harness Run steps provide image, command, environment, secret, output, report, timeout, and failure controls. A reusable template can expose the same governed inputs without requiring a dedicated Ant task type.

The community drone-ant repository does not provide a released integration. At reviewed PR commit 53b582d, PR 1 accepts only goals, uses unpinned Chocolatey packages on LTSC 2022, and its checked-in empty-goals test fails. It does not cover the material Bamboo task inputs.

The CSV says No because no maintained Windows Ant/JDK image and qualified template are currently established.

## 5. The actual gap

Harness can execute Ant today, but the customer lacks a maintained Windows tool environment and a reusable configuration contract. The open plugin PR does not close that gap reliably.

## 6. Recommended Harness solution

Recommendation: include Ant in the shared Windows Java image workstream and provide a reusable Run step template.

The customer configures a pinned image, build file, targets, properties, working directory, reports, and secrets. Engineering adds pinned Ant/JDK packaging, checksums, smoke builds, Windows Kubernetes qualification, and template documentation.

We should not adopt or build an Ant plugin whose principal action is only ant plus arguments. The existing PR should be revisited only if Product requires a Plugin-step user interface, and then it needs substantial hardening.

Result: repeatable Ant execution with Harness-native governance and reporting on the agreed Windows matrix.

## 7. Proposed implementation shape

- Base: same agreed Windows LTSC family as Maven where JDK compatibility allows.
- Contents: pinned JDK and Ant; no project libraries.
- Tags: LTSC, JDK major, Ant version, and image revision.
- Template inputs: image, build file, targets/arguments, properties, ANT_OPTS, workdir, reports, timeout, and failure strategy.
- Project-owned: build.xml, custom tasks, Ivy configuration, and repository dependencies.
- Qualification: a small Ant compile/test fixture, corporate CA/proxy, paths with spaces, secret masking, and failed target.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which Ant and JDK versions are active? | Defines the image and compatibility matrix. | Customer |
| Are custom Ant tasks or Ivy used? | May require private repositories or project-supplied JARs. | Customer |
| Is a native Run template acceptable? | Determines whether the incomplete plugin PR needs evaluation. | Customer / Product |

## 9. Validation plan

Run a representative build.xml on the target Windows Kubernetes node. Exercise custom properties, a path containing spaces, private dependency access, proxy/CA trust, JUnit output, failed targets, cancellation, and secret masking. Record the image digest and exact Ant/JDK versions.

## 10. Dependencies and risks

- Blocking: missing Ant/JDK and custom-task inventory.
- Planning: a dedicated Plugin-step UI would add scope without changing build behavior.
- Implementation: unpinned Chocolatey installation in the existing PR is not acceptable for a supported image.
- Long-term maintenance: Java security updates and old custom Ant task compatibility.

## 11. Planning estimate

1 to 2 engineering weeks is the required planning bucket, including qualification contingency. The development portion is included in one shared engineering week for Maven/Ant/Node/Groovy and is not additive by row. This assumes one actual Ant/JDK pairing and reusable registry/build automation. Hardening drone-ant is separate and conditional if a Plugin-step contract is required.

## 12. What we can tell the customer now

- Harness supports governed Windows Run execution when a compatible Ant/JDK image is supplied; Harness does not currently provide the proposed maintained image.
- The recommended gap closure is a maintained Windows Ant/JDK image and reusable template.
- The open community Ant plugin is not currently release-ready.
- We need the active Ant/JDK versions and any custom task dependencies before confirming the matrix.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 5.

### Bamboo/vendor

- [Atlassian: Ant](https://confluence.atlassian.com/display/BAMBOO/Ant)
- [Atlassian: Configuring a builder task](https://confluence.atlassian.com/bamboo/configuring-a-builder-task-289277037.html)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/development-guides/ci-windows.md
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/RunStepInfo.java
- [harness-community/ci-images at 9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc](https://github.com/harness-community/ci-images/tree/9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc)
- [drone-ant PR 1 commit 53b582d4abfbfb7ffb45561b3d42b7c9f468f310](https://github.com/harness-community/drone-ant/commit/53b582d4abfbfb7ffb45561b3d42b7c9f468f310)
- Repository audit at 53b582d4abfbfb7ffb45561b3d42b7c9f468f310: go test ./... failed the checked-in empty-goals case; go vet ./... and a Windows AMD64 cross-build passed.

Confidence: Medium.
