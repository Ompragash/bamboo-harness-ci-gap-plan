# Artifactory generic resolve and download

| Field | Value |
| --- | --- |
| Bamboo plugin key | bamboo-artifactory-plugin:artifactoryGenericResolveTask |
| Provider | JFrog |
| Customer version(s) | Not provided |
| Harness CSV status | No |
| Scope | CI, existing plugin qualification |
| Recommended Harness approach | Harden and qualify the existing drone-artifactory download command on Windows Kubernetes |
| Solution type | F. Existing plugin extension and repair |
| Discovery required | Yes |
| Planning confidence | Medium |

## 1. What this Bamboo task does

The task resolves files from Artifactory using patterns or a JFrog file spec and downloads them into the Bamboo workspace. JFrog adds authenticated repository access, build/project/module selection, properties, parallelism, and predictable result handling.

## 2. How it works in Bamboo

Bamboo job → JFrog generic resolve task → Artifactory API/CLI with credentials and spec → matched artifacts → local destination and task result.

Material inputs include server, repository/spec, build or project constraints, target path, recursive/flat behavior, properties, retries, and credentials.

## 3. How the customer uses it

Confirmed customer usage: the generic resolve task appears in the inventory. No file specs, source repositories, project/build constraints, destination behavior, or Artifactory version are supplied.

Typical plugin capability: retrieve dependency or build artifacts without embedding raw HTTP logic in the pipeline.

Customer usage context: not confirmed from the available source material.

Smallest question: Provide one exported generic-resolve task, including file spec/pattern, build/project/module selectors, and destination behavior.

## 4. What Harness supports today

drone-artifactory source implements a download command with inline spec or spec path, build, module, project, direct authentication settings, URL, and CA PEM handling. The reviewed repository includes Windows Dockerfiles for multiple LTSC versions, but a supported published Windows release was not verified. Static review found that retries and threads exist in the shared Args structure but are not mapped by the download command, RT proxy mapping is bypassed, invalid combinations can yield an empty successful command list, and no structured Harness output is written.

The CSV says No because this path has not been proven against the customer's Windows Kubernetes and JFrog configuration. It is not evidence that a plugin must be created.

## 5. The actual gap

The gap combines known hardening with a verified field mapping and E2E evidence: proxy wiring, retry/thread behavior if required, command validation, structured downloaded-artifact output if required, Windows paths, file specs, private CA, and no-match/partial-download semantics.

## 6. Recommended Harness solution

Recommendation: repair the mandatory shared RT proxy and command-validation defects, implement download retry/thread/output behavior only when accepted, then qualify and document the existing download command as part of the shared Artifactory workstream.

The customer configures the Plugin step with an image-registry connector, JFrog endpoint/auth settings sourced from Harness secrets, spec/pattern, selectors, and destination. Harness manages secret injection, logs, resources, timeout, and failure strategy. Retry/thread settings and structured outputs are available only after the bounded repair if customer acceptance requires them.

Engineering work is proxy/validation/output and conditional retry/thread hardening, configuration mapping, test fixtures, packaging/release evidence, and Windows E2E tests. We should not create a separate downloader plugin or replace file specs with ad hoc curl commands.

Result: the customer receives the JFrog-aware download behavior in a reusable, governed Harness step.

## 7. Proposed implementation shape

- Existing repository: drone-plugins/drone-artifactory at c5db420e97e7c23ce3723aac30deae5b3a714c1e.
- Existing mapped inputs: inline/file spec, build, module, project, URL, and direct auth settings; CA PEM handling exists in the shared RT path.
- Known repair: RT proxy mapping, invalid command validation, and, if required, download retry/thread flags and a downloaded-artifact manifest output.
- Qualification: target Windows LTSC, path separators and spaces, layout, no-match semantics, checksum, partial failure, large files, CA/proxy, and any added retry/output behavior.
- Outputs: downloaded path list or manifest if currently available; add only if customer acceptance requires a structured output absent from the plugin.
- Packaging: share the same published Windows plugin image and release evidence as Maven/Gradle.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Are Bamboo file specs or simple patterns used? | Defines field mapping and fixtures. | Customer |
| Are build, project, module, properties, or latest selectors used? | Determines semantic parity. | Customer |
| What is expected when nothing matches or one file fails? | Defines task outcome. | Customer |
| Are private CA/proxy and large artifacts present? | Required for representative qualification. | Customer |

## 9. Validation plan

Download a representative file set using the customer's spec on Windows Kubernetes. Verify directory layout, spaces, checksum, no-match and partial-failure behavior, retries, concurrent downloads, private CA/proxy, secret masking, and exact Artifactory metadata. Use a non-production JFrog repository.

## 10. Dependencies and risks

- Blocking: JFrog test tenant and exported task.
- Planning: “latest” selectors can reduce reproducibility.
- Implementation: Windows path behavior, file-spec edge cases, and output contract.
- Long-term maintenance: JFrog CLI and image publication.

## 11. Planning estimate

Included in the shared 1 to 2 engineering week bounded Artifactory hardening and Windows qualification workstream. Do not add a separate row estimate. A broad structured output contract or additional Bamboo semantics require a separate estimate.

## 12. What we can tell the customer now

- Existing Artifactory plugin source includes a generic download path and Windows Dockerfiles; known proxy/validation/output gaps, supported publication, and customer qualification remain open.
- The recommended path is to repair and qualify that codebase rather than create another plugin or rely on raw curl.
- File specs, selectors, and failure semantics must be checked against one exported Bamboo task.
- This work shares packaging and environment testing with Maven/Gradle Artifactory qualification.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 22.

### Bamboo/vendor

- [JFrog: Bamboo Artifactory plug-in](https://docs.jfrog.com/integrations/docs/bamboo-artifactory-plug-in)
- [JFrog: Build integration](https://docs.jfrog.com/artifactory/docs/build-integration)

### Harness

- drone-artifactory at c5db420e97e7c23ce3723aac30deae5b3a714c1e: plugin/plugin.go, plugin/rt_commands.go, plugin/rt_download_cleanup.go, docker/Dockerfile.windows.amd64.ltsc2022
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/PluginStepInfo.java

Confidence: Medium.
