# MSBuild and Visual Studio

| Field | Value |
| --- | --- |
| Bamboo plugin key | plugin.dotnet:msbuild, plugin.dotnet:devenv |
| Provider | Bamboo native .NET plugin |
| Customer version(s) | MSBuild 2.0, 3.5, 4.0, 12, 14, 15, 16, and 17 |
| Harness CSV status | No |
| Scope | CI, Windows image/template and workload discovery |
| Recommended Harness approach | Discover workloads, then qualify the smallest viable Windows Build Tools image and native Run template |
| Solution type | I. Discovery required before selecting implementation |
| Discovery required | Yes |
| Planning confidence | Low |

## 1. What this Bamboo task does

The MSBuild task builds Visual Studio solutions and project files with a chosen MSBuild installation. The devenv variant drives Visual Studio's command-line interface for project types or operations not exposed cleanly through MSBuild.

Bamboo adds capability selection, solution/project path, target, configuration, platform, options, environment, and result handling around Microsoft tooling.

## 2. How it works in Bamboo

Bamboo job → selected MSBuild or Visual Studio capability → solution/project and targets → compiler/toolchain workloads → exit result and generated artifacts/reports.

The selected Visual Studio installation determines targeting packs, SDKs, C++ tools, database tooling, installer tooling, and other workloads. This environment is more material than the wrapper task.

## 3. How the customer uses it

Confirmed customer usage: the inventory spans MSBuild 2.0 through 17 and names both msbuild and devenv tasks. The target platform is Windows Kubernetes.

Typical plugin capability: build SDK-style .NET, full .NET Framework, C++, installer, database, or other Visual Studio project types.

Customer usage context: not confirmed from the available source material.

Smallest question: Which solution/project types, Visual Studio workloads, targeting packs, and devenv-only operations are present in the active plans?

## 4. What Harness supports today

Harness can execute PowerShell commands in a Windows container and manage the image, environment, secrets, outputs, reports, timeouts, and failure strategy. Modern SDK-style projects may use a .NET SDK image and dotnet build.

Harness does not provide a verified Windows container image containing the customer's Visual Studio Build Tools workload matrix. Microsoft supports Build Tools in Windows containers, but full Visual Studio is not supported there. Build Tools installations can be very large, and legacy project types may not work in containers.

The CSV says No because the execution contract exists but the critical toolchain and workload compatibility have not been established.

## 5. The actual gap

The gap is not an MSBuild command wrapper. It is a supportable Windows build environment containing the exact compiler, targeting packs, SDKs, and workloads needed by active projects.

Some legacy or devenv-only builds may require a VM-style runner rather than a Kubernetes Windows container. That choice cannot be made from the version list alone.

## 6. Recommended Harness solution

Recommendation: inventory active project types first, then qualify the smallest viable Visual Studio Build Tools or .NET SDK image and expose it through a reusable PowerShell Run template.

The customer supplies a solution path, targets, configuration, platform, restore behavior, report paths, and required secrets. Harness provides governed execution and reproducible image selection.

Engineering work is an offline or pinned Build Tools layout, exact workload IDs, LTSC compatibility, image-size optimization, smoke solutions, and Windows Kubernetes qualification. We should not build an MSBuild plugin because it cannot supply missing Visual Studio workloads or make an incompatible project container-compatible.

Result: a proven path for the specific POC projects, with unsupported legacy project types identified before commitment.

## 7. Proposed implementation shape

- Supported path: SDK-style .NET using a pinned .NET SDK image when possible.
- Classic path: Windows Server Core plus pinned Visual Studio Build Tools and only required workload/component IDs.
- Legacy path: evaluate customer-managed Windows VM execution if the project needs full Visual Studio, unsupported installers, UI components, or incompatible full-framework behavior.
- Template inputs: solution/project, targets, configuration, platform, restore, properties, workdir, reports, output variables, timeout, and failure strategy.
- Discovery tooling: use vswhere or a fixed installed path, but do not download arbitrary workloads during each build.
- Qualification: exact project fixture, Windows LTSC host match, corporate CA/proxy, paths with spaces, restore, compilation, tests, artifacts, and failure.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which projects are SDK-style, full .NET Framework, C++, installer, database, or devenv-only? | Determines container viability and workload IDs. | Customer |
| Which targeting packs and Windows SDKs are required? | Defines image contents and size. | Customer |
| Which Windows LTSC and CPU architecture are first? | Build Tools and container compatibility vary. | Customer / Engineering |
| Are licensed third-party Visual Studio extensions required? | May block redistribution or unattended installation. | Customer / Vendor |
| Is a Windows VM runner acceptable for non-container-compatible projects? | Provides a fallback for legacy workloads. | Customer / Product |

## 9. Validation plan

Use one representative solution from each active project family. Build it on the target Windows Kubernetes node with exact workloads, private package feeds, proxy/CA, paths with spaces, restore, tests, and artifact output. Verify failure diagnostics, cancellation, secret masking, and image pull/start time. For devenv-only plans, prove Build Tools equivalence or record the need for VM execution.

## 10. Dependencies and risks

- Blocking: project/workload inventory and redistributable installers.
- Planning: the current version list does not reveal project type.
- Implementation: image size, Windows LTSC compatibility, and legacy targeting packs.
- Long-term maintenance: Visual Studio servicing and Windows base-image rebuilds.

## 11. Planning estimate

Discovery required before estimate. A single known SDK-style workload may fit within 1 to 2 engineering weeks. A multi-workload classic Visual Studio matrix, old targeting packs, or VM fallback must be estimated separately after representative projects are available.

## 12. What we can tell the customer now

- Harness supports governed Windows Run execution when a compatible MSBuild/.NET toolchain image is supplied; the proposed maintained Build Tools image is not available today.
- The deciding issue is the Visual Studio workload and project-type matrix, not a missing command wrapper.
- Modern SDK-style projects have the clearest container path.
- We need representative legacy and devenv projects before confirming Windows Kubernetes compatibility.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 6.

### Bamboo/vendor

- [Atlassian: MSBuild](https://confluence.atlassian.com/bamboo0900/msbuild-1167721261.html)
- [Atlassian: Getting started with .NET and Bamboo](https://confluence.atlassian.com/bamboo/getting-started-with-net-and-bamboo-289277288.html)
- [Microsoft: Visual Studio Build Tools workload and component IDs](https://learn.microsoft.com/en-us/visualstudio/install/workload-component-id-vs-build-tools)
- [Microsoft: Visual Studio unsupported operating systems](https://learn.microsoft.com/en-us/troubleshoot/developer/visualstudio/installation/visual-studio-unsupported-operating-systems)
- [Microsoft: Windows container version compatibility](https://learn.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/run-tests/tests-v2.md
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/development-guides/ci-windows.md
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/RunStepInfo.java

Confidence: Low.
