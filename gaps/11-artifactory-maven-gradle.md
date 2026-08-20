# Artifactory Maven and Gradle builds

## Customer need

The customer uses JFrog's Bamboo Maven 3 and Gradle tasks to resolve dependencies from Artifactory, deploy artifacts, and associate module/dependency data with build-info. The active repositories, build identity, Maven/Gradle versions, wrappers, tasks/goals, capture settings, and publish sequence are unknown.

This is a structured repository integration, not only a language build command.

## What Bamboo provides

JFrog's Bamboo plugin selects a Maven/Gradle environment, configures Artifactory resolvers and deployers, runs the build tool, collects dependency/artifact metadata, and publishes or stages build-info according to task configuration.

```text
JFrog Bamboo task
-> tool and Artifactory resolver/deployer configuration
-> Maven or Gradle build
-> artifacts and dependency metadata
-> Artifactory build-info
```

JFrog documentation exposes the task behavior; implementation source for the Bamboo plugin was not located publicly. Exported customer fields are required for exact parity.

## Harness today

`drone-artifactory` already implements Maven, Gradle, JFrog CLI repository commands, build-info operations, direct username/password/token authentication, PEM CA inputs, and Windows Dockerfiles. The LTSC 2022 image currently bundles JFrog CLI 2.115.0, Temurin 17.0.19, Maven 3.9.11, and Gradle 8.13. It supports Maven and Gradle wrapper flags.

Static review at `c5db420e97e7c23ce3723aac30deae5b3a714c1e` found required repairs: Maven/Gradle and other RT command paths can return before applying `enable_proxy`; invalid or empty command/tool combinations can complete without a useful error; structured Harness outputs are absent; Dockerfile downloads are not checksum-verified. The Harness Plugin-step `connectorRef` authenticates the plugin image pull, not JFrog. JFrog credentials still require explicit secret-backed settings.

## Gap

The existing source is the correct base, but it is not yet a supported customer-ready Windows release. Its bundled JDK/Maven/Gradle pair is narrower than the customer estate, and its validation, proxy, release, provenance, and output behavior need repair and qualification.

## Recommended approach

Recommendation: repair and qualify `drone-artifactory`, and align its build-tool selection with the shared Java toolchain strategy instead of creating many Artifactory image variants.

For the POC, use its LTSC 2022 path or a customer-derived image only for the representative supported JDK/tool combination. Prefer `mvnw.cmd` and `gradlew.bat`. If exact Java selection becomes part of the plugin contract, call the shared resolver; do not package three Windows variants merely because Dockerfiles exist.

Required core work: strict command/tool validation, proxy mapping on every path, secret-safe diagnostics, checksum-verified packaging, selected structured outputs, release automation, Windows smoke/E2E tests, and documentation that distinguishes registry connector auth from JFrog credentials.

## POC experience

Proposed plugin inputs, not final Harness YAML:

```yaml
buildTool: mvn
command: build
url: https://company.jfrog.io/artifactory
credentialsSecret: jfrog-ci
useWrapper: true
goals: clean verify
resolverRepositories:
  releases: libs-release
  snapshots: libs-snapshot
deployerRepositories:
  releases: libs-release-local
  snapshots: libs-snapshot-local
buildName: product-ci
buildNumber: <+pipeline.sequenceId>
publishBuildInfo: true
```

Run one successful and one failing Maven or Gradle build against a non-production JFrog project through the customer's proxy/CA path.

## Productized direction

Publish signed Windows images for selected LTSC baselines with a documented JFrog CLI and tool compatibility matrix. Prefer repository wrappers and a shared verified Java resolver for broader supported versions. Add outputs such as build name/number, build-info identifier or URL when the JFrog CLI provides it, artifact summary, and terminal status.

The supported release must have repository ownership, CI tests, SBOM/signing, vulnerability response, and JFrog test-tenant coverage.

## Discovery required

- Export Maven/Gradle task fields, including repositories, build-info flags, env capture, project/module, and publish sequence.
- Which Maven/Gradle/JDK pairs and wrappers block the POC?
- Which JFrog version, authentication, proxy/private CA, and test tenant are available?
- Which outputs or detailed summaries are consumed by later steps?

## Validation

Verify dependency resolution, deployment, Maven and Gradle wrapper paths, direct tool path if active, exact Java identity, build-info modules/dependencies/artifacts, snapshot/release repositories, failed build behavior, proxy/private CA, bad credentials, cancellation, retries, paths with spaces, output values, and secret masking. Compare the published build-info with Bamboo.

## Effort and ownership

- Core repair and one Windows POC qualification: 1 to 2 engineering weeks.
- Product release/lifecycle work may add 1 to 2 weeks depending on existing registry automation.
- Likely ownership: CI + HAR; JFrog is the external compatibility dependency.

## What we can tell the customer

- Harness already has an Artifactory plugin codebase with Maven, Gradle, build-info, CA, and Windows paths.
- The plan is to repair and qualify that plugin, not build a duplicate.
- The POC will target one Windows LTSC and actual build-tool pairing, with wrappers preferred.
- JFrog credentials remain explicit Harness secrets; the plugin image connector is not JFrog authentication.

## Sources

- [JFrog Bamboo Artifactory plugin](https://docs.jfrog.com/integrations/docs/bamboo-artifactory-plug-in)
- [JFrog build-info](https://docs.jfrog.com/integrations/docs/about-build-info)
- [`drone-artifactory` at `c5db420e97e7c23ce3723aac30deae5b3a714c1e`](https://github.com/drone-plugins/drone-artifactory/tree/c5db420e97e7c23ce3723aac30deae5b3a714c1e)
- Harness local evidence: `harness-core` `4b9442f9229a5f33d300dac097e0a1612c92a3ff`, `PluginStepInfo.java`.
