# Artifactory npm and publish build-info

| Field | Value |
| --- | --- |
| Bamboo plugin key | bamboo-artifactory-plugin:artifactoryNpmTask, bamboo-artifactory-plugin:artifactoryPublishBuildInfoTask |
| Provider | JFrog |
| Customer version(s) | npm 5.6.0, 6.4.1, 6.11.3, 6.14.4, 6.14.7, 6.14.8 and additional npm 5/6 variants |
| Harness CSV status | No |
| Scope | CI, existing plugin extension |
| Recommended Harness approach | Candidate drone-artifactory npm/build-info extension after exported-task and vendor-CLI discovery |
| Solution type | F. Existing plugin extension candidate |
| Discovery required | Yes |
| Planning confidence | Medium |

## 1. What this Bamboo task does

JFrog's npm task configures npm resolution/deployment through Artifactory and associates dependency and publication data with a build. The publish build-info task sends the collected build record to Artifactory.

The integration value is authenticated registry setup, build identity, dependency/module metadata, and build-info publication.

## 2. How it works in Bamboo

Bamboo job → JFrog npm task → selected Node/npm with Artifactory registry → install/ci/publish → local build-info collection → publish build-info task → Artifactory build record.

Material inputs include server, resolver/deployer repositories, credentials, npm command, working directory, build name/number, project/module, properties, and publication controls.

## 3. How the customer uses it

Confirmed customer usage: the inventory lists npm 5 and 6 releases and both npm plus publish-build-info tasks. It says nine npm versions but enumerates only six values, so the source itself is incomplete.

Typical plugin capability: install or publish npm packages using Artifactory while recording build metadata.

Customer usage context: not confirmed from the available source material.

Smallest question: Which Node/npm pairings and npm operations are active, and which build-info/project/module fields are populated?

## 4. What Harness supports today

The current drone-artifactory plugin implements build-info and several package/build commands, but no npm command was found at reviewed commit c5db420e97e7c23ce3723aac30deae5b3a714c1e. Its Windows images contain JDK, Maven, and Gradle, not Node.

Harness Run steps can use npm and the JFrog CLI, but the structured registry/build-info behavior belongs in the existing JFrog integration when it is repeated and expected to expose consistent outputs.

The CSV says No because npm workflow support is missing from the plugin and has not been packaged/qualified for Windows.

## 5. The actual gap

The missing contract is an npm command in drone-artifactory that configures resolution/deployment, invokes the agreed npm operation, associates it with build identity, publishes build-info when requested, handles credentials safely, and works on Windows paths.

The listed npm 5/6 versions are end-of-life. Node compatibility must be mapped before selecting image tags.

## 6. Recommended Harness solution

Recommendation: use the existing drone-artifactory repository as the extension point if exported tasks confirm repeated npm resolve/publish/build-info behavior that a bounded vendor-CLI template does not cover adequately.

The customer configures an Artifactory Plugin step with an image-registry connector for pulling the plugin image plus JFrog endpoint/auth settings sourced from Harness secrets. Other settings cover Node image tag, operation, project path, repositories, build identity, project/module, and publication toggle. Harness manages secret injection, logs, timeout, and failure strategy; outputs and command-level retries belong to the plugin contract.

If that gate passes, Engineering adds the npm command, field validation, Windows quoting, build-info sequence, fixtures, and a separate Node-based Windows image variant. We should not add Node to the JVM Artifactory image, and we should not promise every npm 5/6 pairing.

Result: npm resolution/publication and build-info use the same maintained JFrog integration as Maven/Gradle.

## 7. Proposed implementation shape

- Repository: extend drone-artifactory at c5db420e97e7c23ce3723aac30deae5b3a714c1e.
- Command surface: install/ci and publish only if confirmed; resolver/deployer repositories; build name/number; project/module; properties; publish-build-info.
- Authentication: direct JFrog settings backed by Harness secrets, CA PEM and explicit proxy handling, with temporary npm config and cleanup.
- Image: separate Windows Node overlay with plugin binary, jfrog.exe, and one agreed Node/npm pair per tag.
- Outputs: build name, number, project/module, and published build-info URL/identity where the JFrog API supplies it.
- Tests: npm workspace fixture, scoped packages, private registry, failed install, failed build-info publish, proxy/CA, spaces, cleanup, and secret masking.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| What are all nine Node/npm pairings? | npm versions alone do not define a runnable image. | Customer |
| Are install, ci, publish, scoped registries, or workspaces used? | Defines the command contract. | Customer |
| Which build-info/project/module fields are required? | Defines parity and outputs. | Customer |
| Is legacy npm 5/6 required for the POC or can projects upgrade? | Determines support and maintenance policy. | Customer / Product |

## 9. Validation plan

Use a representative package with private and scoped dependencies. On Windows Kubernetes, resolve through Artifactory, publish to a non-production repository if required, publish build-info, and verify module/dependency metadata. Test proxy/CA, paths with spaces, npm config cleanup, bad credentials, failed install, failed build-info publication, retries, and secret masking.

## 10. Dependencies and risks

- Blocking: missing Node/npm pairings and JFrog test tenant.
- Planning: source says nine versions but lists six.
- Implementation: npm config cleanup, Windows quoting, and build-info semantics.
- Long-term maintenance: npm 5/6 and compatible Node releases are end-of-life.

## 11. Planning estimate

1 to 2 engineering weeks after the operation and one initial Node/npm pair are agreed. A multi-version legacy matrix is separate and may exceed this estimate. Packaging/release qualification shares the Artifactory workstream.

## 12. What we can tell the customer now

- If structured npm/build-info behavior is required beyond a bounded vendor-CLI template, the right extension point is the existing Artifactory plugin rather than another integration.
- The npm image should be separate from the Maven/Gradle image.
- The extension can preserve JFrog build-info behavior and Harness-native secret/log controls.
- Exact Node/npm pairings and npm operations are required before confirming scope.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 23.

### Bamboo/vendor

- [JFrog: Bamboo Artifactory plug-in](https://docs.jfrog.com/integrations/docs/bamboo-artifactory-plug-in)
- [JFrog: Set up a build tool with Artifactory](https://docs.jfrog.com/artifactory/docs/set-up-a-build-tool-with-jfrog-artifactory)
- [JFrog: About build-info](https://docs.jfrog.com/integrations/docs/about-build-info)

### Harness

- drone-artifactory at c5db420e97e7c23ce3723aac30deae5b3a714c1e: plugin/plugin.go, plugin/rt_commands.go, plugin/rt_scan_build_info_promote.go, docker/Dockerfile.windows.amd64.ltsc2022
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/PluginStepInfo.java

Confidence: Medium.
