# MSBuild and Visual Studio

## Customer need

The customer lists MSBuild 2.0, 3.5, 4.0, 12, 14, 15, 16, and 17 on Windows. Those numbers do not identify the required SDKs, .NET Framework targeting packs, C++ toolsets, installer/database project support, architecture, or whether full `devenv.exe` behavior is required.

The outcome is a build environment that can reproduce the selected solutions and projects. Visual Studio workload composition and Windows host/container compatibility are more important than a thin MSBuild command wrapper.

## What Bamboo provides

Bamboo selects an installed MSBuild or Visual Studio executable capability, making that tool available only on compatible agents. The MSBuild task accepts a project/solution, options, environment, and working directory. Current Bamboo documentation says it writes project and options to a response file and invokes `msbuild @response-file` by default. The Visual Studio task selects `devenv.exe`, passes solution/options, and can initialize the selected platform through `Vcvarsall.bat`.

```text
Bamboo task
-> agent with installed Build Tools or Visual Studio
-> project, platform, options, environment
-> MSBuild response file or devenv command
```

Bamboo adds executable selection and configuration, but the agent administrator still owns the Microsoft workload installation.

## Harness today

Harness can run PowerShell or command steps in a compatible Windows image or on suitable self-managed infrastructure. It can provide templates, secrets, logs, outputs, timeouts, and reports. It does not currently supply a qualified matrix of Visual Studio Build Tools workloads for this customer's projects.

## Gap

The missing work is workload discovery, a supported environment profile, and representative project qualification. A generic `drone-msbuild` wrapper would not install the correct targeting packs or make `devenv.exe` container-safe. Dynamic per-run Visual Studio installation would be slow, network-dependent, difficult to patch, and often incompatible with container constraints.

## Recommended approach

Recommendation: use immutable, workload-specific Windows Build Tools images for container-compatible projects and a Windows VM/runner lane for full Visual Studio or unsupported workloads.

| Option | Assessment |
| --- | --- |
| Fixed workload images | Preferred for known MSBuild/SDK/Build Tools profiles; large but testable and reproducible. |
| One broad Visual Studio image | Avoid by default; very large, high patching load, and conflicting legacy workloads. |
| Dynamic plugin provisioning | Rejected for Visual Studio installation during each build. A thin command contract may still normalize inputs. |
| Hybrid | Preferred: a few maintained profiles plus customer image/VM exceptions. |
| Customer image | Fastest POC and appropriate for unsupported legacy workloads. |

## POC experience

Select one representative solution and use the customer's existing Build Tools image or runner. Configure a governed template with project/solution, target, configuration, platform, properties, environment, response-file behavior, and report/artifact paths.

Proposed template inputs, not final Harness YAML:

```yaml
tool: msbuild
solution: src/Product.sln
configuration: Release
platform: x64
targets: [Restore, Build]
properties:
  ContinuousIntegrationBuild: "true"
```

If the project requires `devenv.exe`, desktop interaction, unsupported installer components, or a workload that cannot run correctly in Windows containers, the POC uses a VM runner rather than claiming container parity.

## Productized direction

Maintain a small set of named workload profiles, each with an exact Windows/LTSC compatibility statement, Build Tools channel/version, component list, servicing test, SBOM, signing, and rebuild policy. A reusable template can preserve Bamboo's project/options/environment UX and response-file safety. Do not make Visual Studio installation part of the shared lightweight runtime resolver.

Legacy profiles move to best-effort or customer-provided support. Full Visual Studio licensing and `devenv.exe` use require separate legal and infrastructure validation.

## Discovery required

- Which solution/project types and Microsoft workload/component IDs are POC blockers?
- Which .NET Framework targeting packs, Windows SDKs, C++ toolsets, installer/database projects, and architectures are required?
- Which builds require `devenv.exe` rather than MSBuild or `dotnet build`?
- What does the customer's label “Visual Studio 2025” mean?

## Validation

Build representative projects on the target host and container isolation mode. Verify restore, compile, generated artifacts, custom targets, native toolchains, paths with spaces, response-file quoting, private package access, failure propagation, cancellation, and NUnit/MSTest handoff. Compare binary/artifact metadata and logs with Bamboo where deterministic equality is expected.

## Effort and ownership

- Discovery before estimate for the full estate.
- POC: 1 to 2 engineering weeks for one known workload profile after inputs and installers exist.
- Productization: 2 to 4 engineering weeks per bounded profile family, plus ongoing Microsoft servicing.
- Likely ownership: CI + Platform; Microsoft licensing and vendor support are external dependencies.

## What we can tell the customer

- Harness can govern MSBuild and Visual Studio command execution on compatible Windows infrastructure.
- The POC will qualify the project workload, not infer compatibility from an MSBuild version number.
- Visual Studio workloads will use prebuilt profiles or a VM, not an installer downloaded on every build.
- We need representative project types and component IDs before committing to coverage.

## Sources

- [Atlassian MSBuild task](https://confluence.atlassian.com/bamboo0900/msbuild-1167721261.html)
- [Atlassian Visual Studio task](https://confluence.atlassian.com/bamboo/visual-studio-289277041.html)
- [Microsoft Build Tools workload and component IDs](https://learn.microsoft.com/en-us/visualstudio/install/workload-component-id-vs-build-tools)
- [Microsoft Windows container version compatibility](https://learn.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility)
- Harness local evidence: `developer-hub` `1c7c98f1d76bb7b8330d6ffba96f984878a32748`, Windows CI and test docs.
