# Maven

| Field | Value |
| --- | --- |
| Bamboo plugin key | plugins.maven:task.builder.mvn |
| Provider | Bamboo native |
| Customer version(s) | All possible Maven versions with JDK 7, 8, 11, 17, and 21 |
| Harness CSV status | No |
| Scope | CI, Windows image and reusable template |
| Recommended Harness approach | Maintained Windows Maven/JDK image family with native Run and Test steps |
| Solution type | C. Language or tool image |
| Discovery required | Yes |
| Planning confidence | Medium |

## 1. What this Bamboo task does

The task runs Maven goals against a Java repository. Bamboo adds tool-capability selection, working-directory and environment configuration, failure handling, and optional test-result collection around the Maven invocation.

Teams use it for lifecycle phases such as clean, test, package, and deploy. Maven itself remains responsible for resolving the project model, dependencies, plugins, and build lifecycle.

## 2. How it works in Bamboo

Bamboo job → Maven task → selected Maven/JDK capability → pom.xml or wrapper → Maven exit status and generated reports.

Material inputs include the JDK and Maven capability, goals, POM location, working directory, environment variables, and command-line options. A non-zero Maven result fails the task. Bamboo can collect JUnit reports produced by the build.

## 3. How the customer uses it

Confirmed customer usage: the inventory asks for Maven across JDK 7, 8, 11, 17, and 21 and says all possible Maven versions. The email also establishes Windows Kubernetes as the target execution model.

Typical plugin capability: Maven compile, test, package, publish, and plugin goals with a selected JDK and Maven installation.

Customer usage context: not confirmed from the available source material.

Smallest question: Which Maven versions, goals, wrapper usage, and JDK-to-project combinations appear in the exported Bamboo plans?

## 4. What Harness supports today

Harness CI has native Run and Test step contracts that manage the image, command, environment, secrets, outputs, reports, timeouts, and failure behavior. The Windows Kubernetes documentation confirms Run steps and cache steps on supported Windows build infrastructure. Cache Intelligence can use customer-defined Windows paths. Test Intelligence is available for supported Java test frameworks, subject to its current platform matrix.

The CSV says No because Harness does not currently publish a maintained Windows Maven/JDK image matrix matching this inventory. The community ci-images repository contains Linux images only at reviewed commit 9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc.

## 5. The actual gap

The execution step exists. The missing productized layer is a versioned, patched, Windows-compatible Maven/JDK environment with a clear support policy and representative Windows Kubernetes qualification.

JDK 7 is the largest compatibility and maintenance issue. It is end-of-life and may require a time-bounded migration image rather than inclusion in a normal supported matrix.

## 6. Recommended Harness solution

Recommendation: publish a small Windows Maven/JDK image family and use it through native Run and Test steps with a reusable template.

The customer selects a pinned image tag, Maven goals, POM or wrapper path, reports, cache paths, and secrets. Harness continues to own credentials, logs, outputs, reports, timeout, failure strategy, RBAC, and template governance.

Engineering work is image source, pinned downloads and checksums, LTSC-specific build manifests, smoke projects, vulnerability rebuild policy, and a reusable step template. Prefer the repository's mvnw.cmd so the project owns its Maven version. We should not build a Maven plugin whose only behavior is invoking Maven.

Result: a governed Maven experience on the agreed Windows/JDK combinations, with native reports and optional Test Intelligence where supported.

## 7. Proposed implementation shape

- Base OS: one agreed Windows Server Core LTSC baseline for the POC; expand only after host/image compatibility is agreed.
- Image family: JDK plus Maven fallback. Keep JDK major versions separate or use clearly pinned tags.
- Tags: immutable LTSC, JDK, Maven, and image revision components; no floating all-versions promise.
- Project-owned items: Maven wrapper, pom.xml, settings structure, goals, build plugins, and application dependencies.
- Harness template: image, goals, POM path, working directory, settings secret, cache paths, report paths, outputs, timeout, and failure strategy.
- Qualification: AMD64 Windows Kubernetes first. ARM64 remains a separate feasibility decision because the inventory mentions it but current Windows tooling availability is not proven.
- Source location: harness-community/ci-images can hold the sources, but its current lifecycle is marked TBU and requires an explicit owner and release process.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which Maven versions and goals are actually used? | Defines the smallest support and smoke-test matrix. | Customer |
| Do repositories contain mvnw.cmd? | A wrapper removes most Maven-version packaging requirements. | Customer |
| Is JDK 7 needed for the POC or only historical plans? | Determines whether a legacy exception image is required. | Customer / Product |
| Which LTSC and CPU architecture are required first? | Windows host/image compatibility and binary availability differ. | Customer / Engineering |

## 9. Validation plan

Run representative customer projects on the target Windows Kubernetes node using paths with spaces, repository wrappers where present, corporate proxy and CA settings, private repository credentials, dependency cache restore/save, and JUnit report ingestion. Prove success and Maven failure behavior, secret masking, cancellation, and output capture. Record the exact image digest and tool versions.

## 10. Dependencies and risks

- Blocking: no representative project or agreed version subset.
- Planning: “all possible versions” cannot be a maintainable acceptance criterion.
- Implementation: Windows base-image compatibility, proxy/CA trust, and large image pulls.
- Long-term maintenance: JDK 7 and old Maven releases require a separate legacy support decision.

## 11. Planning estimate

1 to 2 engineering weeks is the required planning bucket, including qualification contingency. The development portion is included in one shared engineering week for Maven/Ant/Node/Groovy and is not additive by row. This assumes one actual Maven/JDK pairing and reusable registry/build automation. JDK 7, broad JDK coverage, production lifecycle, multi-LTSC, and ARM64 are not included.

## 12. What we can tell the customer now

- Harness supports governed Windows Run/Test execution when a compatible Maven/JDK image is supplied; Harness does not currently provide the proposed maintained image matrix.
- The planned gap is a maintained Windows Maven/JDK environment, not a new Maven integration.
- We need the real Maven/JDK matrix and initial Windows LTSC target before confirming compatibility.
- JDK 7 needs an explicit legacy migration decision.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 3.

### Bamboo/vendor

- [Atlassian: Configuring a builder task](https://confluence.atlassian.com/bamboo/configuring-a-builder-task-289277037.html)
- [GitHub Actions: Building and testing Java with Maven](https://docs.github.com/en/actions/tutorials/build-and-test-code/java-with-maven)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/development-guides/ci-windows.md
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/run-tests/tests-v2.md
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/caching-ci-data/cache-intelligence.md
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/RunStepInfo.java and TestStepInfo.java
- [harness-community/ci-images at 9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc](https://github.com/harness-community/ci-images/tree/9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc)

Confidence: Medium.
