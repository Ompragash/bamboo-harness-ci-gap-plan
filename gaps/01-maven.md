# Maven

## Customer need

The customer runs Maven builds on Windows with JDK 7, 8, 11, 17, and 21. The Harness replacement must work in Kubernetes Windows containers and cover the POM, goals, profiles, arguments, `MAVEN_OPTS`, settings, working directory, environment, and test reports.

## How Bamboo handles it

Bamboo administrators install JDK and Maven versions on long-lived agents and register their paths as capabilities. The Maven task selects a JDK label and Maven executable label; both become requirements inherited by the job. Bamboo schedules the job only on an agent with matching capabilities.

The task then constructs the Maven invocation and configures environment, working directory, exit handling, and test-result paths. Java and Maven are normally already present on the selected agent.

```text
Bamboo task requires JDK 17 + Maven 3.x
-> Bamboo selects a matching Windows agent
-> task constructs the Maven command
-> installed JDK and Maven execute
-> logs, status, and reports return to Bamboo
```

## Harness implementation

Recommendation: use a Harness-maintained Windows Java build image through a native Run step. A dedicated Maven plugin is not required for the POC.

The pipeline explicitly references the required image tag, for example:

```text
harness/windows-java-build:temurin17
```

The standard image family has separate tags for Temurin 8, 11, 17, and 21. Each tag contains one JDK, PowerShell, certificates/base utilities, and one Harness-supported Maven fallback. Maven, Ant, and Groovy share this image family because the build-tool distributions are small compared with the Windows base and JDK. Exact tags must also identify the supported Windows LTSC and tool release in Harness's release metadata.

```text
Harness Run step
-> explicit harness/windows-java-build:temurin17 image
-> .\mvnw.cmd clean verify, or bundled mvn clean verify
-> Harness logs, outputs, failure strategy, and JUnit report paths
```

The repository Maven Wrapper remains supported when it is present. The wrapper downloads its configured Maven distribution when it is absent from `~/.m2/wrapper/dists`; therefore it is not treated as a zero-download mechanism in an ephemeral container. The image includes a standard Maven release. A repository that requires another Maven release can use its wrapper with an approved mirror and checksum, or justify a bounded additional image tag when repeated download is unacceptable.

A single plugin image containing all four JDKs is not recommended. Current Temurin Windows x64 JDK ZIPs for 8, 11, 17, and 21 total about 702 MB before extraction, the Windows base, Maven, and plugin layers. Separate image tags keep each pull and vulnerability surface bounded.

The Run step is still a native Harness capability: Harness controls the image, connector, secrets, environment, logs, timeout, output variables, reports, and failure strategy. A reusable Run Step Template provides the approved command pattern without adding plugin code whose main job would only be to assemble a Maven CLI.

JDK 7 is not included in the standard family. It requires an explicit Harness support decision covering distribution rights, security servicing, and Windows-container qualification. If Harness cannot maintain it safely, it is unsupported for the POC.

## What we still need to confirm

- Is JDK 7 a hard POC blocker?
- Which exact JDK and Maven combinations are active rather than theoretically possible?
- Which repositories use `mvnw.cmd`, and can wrapper downloads use an approved mirror and checksum?
- Which Windows LTSC version and CPU architecture are the POC target?

## Customer position

- Maven uses a native Harness Run step with an explicit Harness-owned image tag.
- Harness maintains separate Java 8, 11, 17, and 21 tags; no multi-JDK plugin image is proposed.
- Common JDK and Maven tooling is present before the step starts.
- JDK 7 requires an explicit support decision.

## Sources

- [Bamboo Maven task](https://confluence.atlassian.com/bamboo1200/maven-1680480796.html)
- [Bamboo capabilities and requirements](https://confluence.atlassian.com/bamboo/about-capabilities-and-requirements-289277171.html)
- [Harness Run step](https://developer.harness.io/docs/continuous-integration/use-ci/run-step-settings/)
- [Harness Plugin step](https://developer.harness.io/docs/continuous-integration/use-ci/use-drone-plugins/run-a-drone-plugin-in-ci/)
- [Apache Maven Wrapper behavior](https://maven.apache.org/tools/wrapper/index.html)
- [Eclipse Temurin supported platforms](https://adoptium.net/supported-platforms)
