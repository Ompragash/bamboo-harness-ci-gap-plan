# Maven

## Customer need

The customer runs Maven builds on Windows using JDK 7, 8, 11, 17, and 21. Harness must provide the same structured build experience on Kubernetes Windows worker nodes, where every step starts in a Windows container.

The Maven task must support the selected Java profile, Maven Wrapper or Harness-supported Maven, POM path, goals, profiles, arguments, `MAVEN_OPTS`, working directory, environment variables, and test report paths.

## How Bamboo handles it

Bamboo relies on long-lived Windows agents. An administrator installs JDK and Maven versions and registers them as Bamboo capabilities. The Maven task adds agent requirements, so Bamboo schedules the job on an agent that already has the selected JDK and Maven.

The task then turns POM, goals, profiles, environment, working directory, and report settings into the Maven invocation. Bamboo does not normally install Java or Maven for every build.

```text
Bamboo selects Windows agent with JDK + Maven already installed
-> Maven task receives POM, goals, profiles, and options
-> installed Maven runs
-> exit status and test reports return to Bamboo
```

## Harness implementation

Recommendation: build a Harness Maven plugin backed by Harness-maintained Windows Java runtime images.

Harness should initially maintain Temurin Java 8, 11, 17, and 21 runtime profiles for the selected Windows LTSC baseline. Temurin publishes Windows x64 binaries for these LTS versions under GPLv2 with the Classpath Exception. Harness must pin exact builds, rebuild for security updates, publish signed images, and test each supported profile on Windows Kubernetes.

The user selects a supported Java profile and Maven inputs. The Harness step abstraction resolves the internal runtime image, so the user does not choose an image tag.

```text
Harness Maven Plugin
-> Harness Windows Java 17 runtime
-> repository mvnw.cmd, or Harness-supported Maven fallback
-> clean verify
-> Harness logs, outputs, and test results
```

The plugin prefers `mvnw.cmd` because the repository then owns its Maven version. If no wrapper exists, each Java runtime includes one Harness-supported Maven fallback. Additional Maven versions are added only when an active customer build requires them. This creates four logical Java profiles, not a Java x Maven image matrix, and does not download a JDK during the pipeline.

Proposed plugin inputs: Java profile, wrapper/fallback selection, POM, goals, profiles, arguments, settings secret, `MAVEN_OPTS`, working directory, environment, and report paths.

JDK 7 is not part of the standard runtime set. Public Temurin releases do not provide JDK 7. Azul lists Java 7 support on Windows Server, but redistribution, commercial support, security updates, and Windows-container testing require an explicit Harness support decision before it can be included.

## What we still need to confirm

- Is JDK 7 a hard POC requirement?
- Do any builds require a Java distribution other than Temurin?
- Which repositories use `mvnw.cmd`, and which Maven fallback versions are required?
- Which Windows LTSC version and CPU architecture are the POC target?

## Customer position

- Harness can provide a structured Maven task on Windows Kubernetes.
- Java 8, 11, 17, and 21 will be prebuilt, secured, and maintained by Harness once the target LTSC is selected.
- The Maven plugin hides internal image selection and prefers the repository wrapper.
- JDK 7 requires an explicit Harness support decision and is not included in the standard runtime set today.

## Sources

- [Atlassian Maven task](https://confluence.atlassian.com/bamboo1200/maven-1680480796.html)
- [Bamboo Specs Maven reference](https://docs.atlassian.com/bamboo-specs-docs/10.0.2/specs.html?yaml=)
- [Eclipse Temurin supported platforms](https://adoptium.net/supported-platforms)
- [Eclipse Temurin licensing and availability](https://adoptium.net/docs/faq)
- [Azul supported platforms](https://docs.azul.com/core/supported-platforms)
