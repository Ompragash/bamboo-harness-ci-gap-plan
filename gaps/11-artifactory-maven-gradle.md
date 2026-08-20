# Artifactory Maven and Gradle build

| Field | Value |
| --- | --- |
| Bamboo plugin key | bamboo-artifactory-plugin:maven3Task, bamboo-artifactory-plugin:artifactoryGradleTask |
| Provider | JFrog |
| Customer version(s) | Customer Maven matrix; Gradle version not provided |
| Harness CSV status | No |
| Scope | CI, existing plugin qualification |
| Recommended Harness approach | Harden and qualify the existing drone-artifactory Maven/Gradle paths on Windows Kubernetes |
| Solution type | F. Existing plugin extension and repair |
| Discovery required | Yes |
| Planning confidence | Medium |

## 1. What this Bamboo task does

JFrog's Bamboo tasks run Maven or Gradle with Artifactory configured as resolver and deployer, then collect and publish build-info. The integration adds repository configuration, authentication, module/dependency metadata, environment capture, and build identity around the build tool.

This is more than a generic build command because Artifactory build-info and repository resolution are structured integration behavior.

## 2. How it works in Bamboo

Bamboo job → JFrog Maven/Gradle task → selected build tool plus Artifactory configuration → dependency resolution/build/deploy → JFrog build-info publication → task result.

Material inputs include Artifactory server, resolver/deployer repositories, credentials, build name/number, modules, POM/build file, goals/tasks, properties, and build-info publication options.

## 3. How the customer uses it

Confirmed customer usage: the inventory includes Maven and Gradle Artifactory build tasks. Maven follows the broad JDK/Maven estate; no Gradle versions, repositories, file specs, build-info fields, or environment constraints are provided.

Typical plugin capability: resolve and deploy packages while publishing Artifactory build metadata.

Customer usage context: not confirmed from the available source material.

Smallest question: Which Maven/Gradle task fields, repositories, build-info operations, wrappers, and JFrog server version are present in the exported plans?

## 4. What Harness supports today

The existing drone-artifactory repository source implements Maven and Gradle commands, dependency/build-info behavior, direct authentication settings, CA PEM handling, thread settings, project/module metadata, scan, promotion, and cleanup.

At reviewed commit c5db420e97e7c23ce3723aac30deae5b3a714c1e it also has Windows 1809, LTSC 2022, and LTSC 2025 Dockerfiles containing pinned JFrog CLI 2.115.0, Temurin JDK 17.0.19, Maven 3.9.11, and Gradle 8.13. This proves Windows packaging source exists, but a supported published Windows release, ownership, SBOM/signing, and release qualification were not verified. Static review also found that the RT command path returns before PLUGIN_ENABLE_PROXY maps Harness proxy variables, has no command-level retry mapping, can accept invalid command/tool combinations without a validation error, and writes no structured Harness output.

The CSV says No because a field-level mapping and customer-environment Windows Kubernetes qualification have not been established, not because the integration is absent.

## 5. The actual gap

The gap combines known hardening with customer qualification: proxy wiring for RT commands, command/tool validation, structured outputs if required, command-level retry semantics if required, Windows quoting, JFrog endpoints, wrappers/tool versions, private CA, and Bamboo field mapping.

The bundled JDK/Maven/Gradle versions are fallbacks. Customer repositories should use mvnw.cmd and gradlew.bat when possible to avoid forcing every historical build-tool version into the plugin image.

## 6. Recommended Harness solution

Recommendation: repair the mandatory RT proxy and command-validation defects, implement only the accepted retry/output contract, then qualify the existing drone-artifactory plugin on the agreed Windows Kubernetes baseline.

The customer configures a Harness Plugin step with an image-registry connector for pulling the plugin image plus JFrog endpoint/auth settings sourced from Harness secrets. Other settings cover build tool, wrapper or executable, goals/tasks, repositories, build identity, modules, project, CA/proxy, and desired build-info behavior. Harness manages the step lifecycle, secret injection, logs, timeout, and failure strategy; plugin outputs and command-level retries must be implemented explicitly.

Engineering work is the known proxy/validation/output hardening, configuration mapping, image-digest selection, E2E fixtures, certificate/proxy qualification, documentation, and release evidence. We should not build a second Artifactory plugin. Result: one integration codebase covers Maven/Gradle and shares packaging and test infrastructure with the generic-download row.

## 7. Proposed implementation shape

- Existing repository: drone-plugins/drone-artifactory at c5db420e97e7c23ce3723aac30deae5b3a714c1e.
- Already implemented: Maven, Gradle, download, dependencies, build-info, promotion, cleanup, direct auth settings, CA PEM handling, and Windows-aware paths.
- Known repair: RT proxy mapping, invalid command/tool validation, structured outputs if accepted, and command-level retry behavior if required.
- Preferred build tools: repository wrappers; pinned image fallbacks for agreed versions.
- Qualification matrix: one LTSC baseline first, Maven and Gradle fixtures, Windows paths, private CA, proxy, build-info, module/project fields, retry/failure behavior, and outputs.
- Potential extension gate: only missing Bamboo fields proven active in exported tasks.
- Packaging: use current Windows images as evidence, but confirm publication, registry ownership, signatures/SBOM, and supported tags.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which JFrog server version, repositories, and auth model are used? | Defines compatibility and tenant setup. | Customer |
| Are mvnw.cmd and gradlew.bat used? | Determines tool-version packaging needs. | Customer |
| Which build-info fields, modules, scans, promotion, or cleanup operations are required? | Defines qualification versus extension. | Customer |
| Are private CA and outbound proxy mandatory? | Required for Windows E2E acceptance. | Customer |

## 9. Validation plan

Run representative Maven and Gradle projects on the target Windows Kubernetes node. Resolve private dependencies, deploy a non-production package if in scope, publish build-info, verify modules/dependencies/environment masking, exercise wrappers and paths with spaces, and test private CA, proxy, bad credentials, retryable network failure, and duplicate build numbers. Compare resulting Artifactory build metadata with Bamboo output.

## 10. Dependencies and risks

- Blocking: JFrog test tenant, customer task exports, and non-production credentials.
- Planning: the bundled JDK 17/tool versions do not cover the stated Java estate.
- Implementation: docs/input-name drift, Windows quoting, CA/proxy, and registry publication.
- Long-term maintenance: JFrog CLI and Windows base-image updates.

## 11. Planning estimate

1 to 2 engineering weeks for the shared bounded Artifactory hardening and Windows qualification workstream, assuming one LTSC baseline, a JFrog test tenant, representative Maven/Gradle projects, and no broad output schema. Count this once with generic download. Additional Bamboo semantics require a separate estimate.

## 12. What we can tell the customer now

- Existing Artifactory plugin source implements the core Maven and Gradle workflows and includes Windows Dockerfiles; known proxy/validation/output gaps, supported publication, and customer qualification remain open.
- The current need is to harden and qualify that codebase, not create a second integration.
- Repository wrappers are the preferred way to preserve project-specific Maven and Gradle versions.
- We need the exported JFrog task fields and a test tenant before confirming field coverage.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 20.

### Bamboo/vendor

- [JFrog: Bamboo Artifactory plug-in](https://docs.jfrog.com/integrations/docs/bamboo-artifactory-plug-in)
- [JFrog: About build-info](https://docs.jfrog.com/integrations/docs/about-build-info)
- [JFrog: Maven Artifactory plugin](https://docs.jfrog.com/integrations/docs/maven-artifactory-plugin)
- [JFrog: Gradle Artifactory plugin](https://docs.jfrog.com/integrations/docs/artifactory-gradle-plugin)

### Harness

- drone-artifactory at c5db420e97e7c23ce3723aac30deae5b3a714c1e: plugin/plugin.go, plugin/rt_commands.go, plugin/mvn.go, plugin/gradle.go, docker/Dockerfile.windows.amd64.ltsc2022
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/PluginStepInfo.java

Confidence: Medium.
