# Maven

## Customer need

The customer needs Maven builds on Windows across JDK 7, 8, 11, 17, and 21. The inventory says all Maven versions, but no active Maven/JDK pairing, distribution, wrapper use, goal set, or settings configuration has been exported yet.

The required outcome is not only that `mvn` starts. Teams need a predictable way to select Java and Maven, choose a POM and working directory, pass goals and environment, use corporate repositories, cache dependencies, publish reports, and understand which legacy combinations Harness supports.

## What Bamboo provides

Bamboo's native Maven task selects installed Maven and JDK capabilities. Those selections become agent requirements. It accepts goals and profiles, POM override, environment and `MAVEN_OPTS`, working subdirectory, and standard or custom JUnit result patterns. It can use the Maven return code rather than its legacy log parser. Maven local-repository isolation is configured on the Maven executable capability.

```text
Bamboo task
-> agent with selected Maven and JDK capabilities
-> POM, goals, environment, working directory
-> Maven build and optional JUnit results
```

Bamboo does not prove that every replacement must dynamically download Maven. Its core customer value is validated toolchain selection and a reusable Maven-oriented configuration surface.

## Harness today

Harness Run and Test steps already provide Windows execution, images, environment, secrets, logs, timeouts, failure strategies, outputs, and report ingestion. A repository `mvnw.cmd` can own the Maven distribution. Harness Cache Intelligence can cache project dependencies, but it is not a verified runtime distribution cache.

`harness-community/ci-images` has no Windows lane at the reviewed commit. The indexed community `drone-java-maven-plugin` is a stale 2022 Linux-only shell wrapper fixed to Maven 3.8.5 and JDK 11; it does not select Java/Maven versions and can print generated settings containing credentials in debug mode. It is not a Windows product base.

## Gap

Harness lacks a supported Windows Maven experience that separates JDK distribution/version from Maven selection, prefers repository wrappers, supplies controlled fallback provisioning, and defines support for legacy combinations. A bare command or a fixed image tag does not preserve Bamboo's useful capability-selection experience across this estate.

Capability mapping:

| Bamboo capability | Customer-used behavior to confirm | Harness primitive | Missing abstraction |
| --- | --- | --- | --- |
| Maven executable capability | Maven or wrapper selection | Image, Run/Plugin step | Validated wrapper/version policy |
| JDK capability | Distribution and version | Image/environment | Supported Java resolver and support class |
| POM, goals, profiles, options | Repeatable build configuration | Step inputs/template | Maven-oriented input contract |
| Settings, environment, local repository | Corporate resolution and isolation | Secrets, volumes/cache | Safe settings/mirror/cache convention |
| Test result patterns | Visible build results | Test reports | Qualified defaults and mappings |

## Recommended approach

Recommendation: use a wrapper-first Maven contract backed by a shared secure Java/tool resolver, with a customer-supplied toolchain image as the POC fallback.

| Design | Assessment |
| --- | --- |
| 1. Fixed Maven/JDK images | Fast and reproducible, but creates a tag and patching matrix across Maven, JDK, distribution, and LTSC. Suitable only for a few supported common pairings. |
| 2. Plugin selects JDK and Maven | Best structured UX and version breadth, but runtime source, verification, proxy, cache, and offline behavior become product responsibilities. |
| 3. Wrapper-first plugin plus independent JDK | Preferred default. The repository pins Maven while Harness selects Java. It reduces the tool matrix and follows Maven's repository-owned wrapper model. |
| 4. Hybrid prebuilt plus provisioning | Preferred runtime implementation. Prepackage common supported Java lines, resolve other approved versions from a verified cache/mirror, and use customer images for exceptions. |
| 5. Customer image plus Maven wrapper plugin | Fastest POC and handles licensed/EOL combinations, but customer effort and inconsistent support make it insufficient as the only product answer. |

JDK 7 must be customer-provided legacy or come from a customer-approved vendor mirror. Oracle's archive requires an account and has license constraints; a current general Temurin JDK 7 distribution should not be assumed. Azul or another vendor may be possible only after commercial and security approval.

## POC experience

Use one customer-approved Windows image for each representative POC pairing, or a single image containing only the few POC runtimes, and a governed Maven template. Prefer `mvnw.cmd`; use a pinned Maven already present in the image only where no wrapper exists.

Proposed user-facing inputs, not final Harness YAML:

```yaml
java:
  distribution: temurin
  version: "17"
maven:
  executable: wrapper
pom: pom.xml
goals: [clean, verify]
profiles: [ci]
settingsSecret: maven-settings
reports: [target/surefire-reports/*.xml]
```

Harness still manages the step, secret references, logs, reports, timeout, retry/failure strategy, and audit. The command is the project build engine inside that managed contract, not an unmanaged script.

## Productized direction

Create a Maven-oriented plugin or first-class contract that calls a shared resolver. It should support exact Java distribution/version, wrapper-first Maven, an exact Maven fallback, POM, goals, profiles, settings, environment, working directory, local repository/cache convention, and report paths. It must support approved mirrors, proxy/private CA, deterministic checksums or signatures, bounded retry, safe logs, and offline errors.

Use three support classes: Harness-supported current combinations, compatibility or best effort through the generic resolver, and customer-provided legacy. Do not promise every cross-product of Maven, JDK, distribution, LTSC, and architecture.

## Discovery required

- Which Maven/JDK pairs actually block the POC, and which JDK distribution is used for each?
- Which repositories contain `mvnw.cmd`, and are wrapper distributions mirrored internally?
- Which POM, settings, toolchains, profiles, local repository, and report options are active?
- Must all runtime archives come through an internal mirror, proxy, or private CA?
- Which JDK 7 binary is legally and operationally approved?

## Validation

Run one representative project for each selected support class on the target LTSC node. Verify exact Java/Maven identity, wrapper and no-wrapper paths, private parents/plugins, settings and CA, paths with spaces, cache cold/warm behavior, passed and failed tests, report counts, cancellation, offline failure, checksum rejection, and secret masking. Compare goals, artifacts, and test results with the Bamboo execution.

## Effort and ownership

- POC: 1 to 2 engineering weeks within the shared Windows toolchain workstream, assuming customer images and representative projects exist.
- Productized Maven contract: 2 to 4 engineering weeks after input validation, excluding the shared resolver foundation.
- Shared resolver foundation: a separate 2 to 4 week bounded workstream.
- Likely ownership: CI with Platform for runtime provenance, cache, and release operations.

## What we can tell the customer

- Harness can run and report Maven builds on Windows today with a compatible toolchain image.
- The POC will prefer repository Maven wrappers and qualify the actual Java/Maven pairs, not build an all-version image matrix.
- The long-term direction is selectable, verified toolchains with explicit support classes.
- JDK 7 requires an approved customer or vendor distribution before it can be committed.

## Sources

- [Atlassian Maven task](https://confluence.atlassian.com/bamboo1200/maven-1680480796.html)
- [Bamboo Specs Maven task reference](https://docs.atlassian.com/bamboo-specs-docs/10.0.2/specs.html?yaml=)
- [GitHub `setup-java`](https://github.com/actions/setup-java/blob/main/README.md)
- [Oracle Java SE 7 archive](https://www.oracle.com/java/technologies/javase/javase7-archive-downloads.html) and [Java SE 7 license](https://www.oracle.com/downloads/licenses/javase7-license.html)
- [`drone-java-maven-plugin` at `f72fbd12e522cd70d73a1aac58c2c95fa41a57c5`](https://github.com/kameshsampath/drone-java-maven-plugin/tree/f72fbd12e522cd70d73a1aac58c2c95fa41a57c5)
- [`ci-images` at `9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc`](https://github.com/harness-community/ci-images/tree/9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc)
- Harness local evidence: `developer-hub` `1c7c98f1d76bb7b8330d6ffba96f984878a32748`, Windows CI, Run/Test, reports, and cache docs.
