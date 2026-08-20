# MSBuild and Visual Studio

## Customer need

The customer lists MSBuild 2.0, 3.5, 4.0, 12, 14, 15, 16, and 17. Harness must build the selected solutions and projects in Windows containers on Kubernetes, including the required SDKs, .NET Framework targeting packs, C++ toolsets, and other Build Tools workloads.

The implementation must state which workloads are supported in Windows containers. It cannot assume that every historical Visual Studio installation or `devenv.exe` workflow can be moved unchanged.

## How Bamboo handles it

Bamboo administrators install MSBuild or Visual Studio on long-lived Windows agents and register executable capabilities. The MSBuild and Visual Studio tasks require those capabilities, so Bamboo selects a matching agent.

The MSBuild task supplies a solution/project, options, environment, and working directory, then normally invokes MSBuild through a response file. The Visual Studio task can invoke `devenv.exe` and initialize platform variables through Visual Studio developer scripts.

```text
Bamboo selects agent with required Visual Studio/MSBuild installation
-> task supplies project, targets, configuration, platform, and options
-> installed MSBuild or devenv runs
-> result returns to Bamboo
```

## Harness implementation

Recommendation: build a Harness MSBuild plugin backed by a small Harness-maintained Windows .NET and Visual Studio Build Tools runtime family.

The initial runtime profiles should be:

- a Windows .NET SDK profile for modern SDK-style projects;
- a Windows .NET Framework SDK profile for supported full-framework builds;
- a Visual Studio Build Tools 2022 profile for MSBuild 17 and the customer-required container-compatible workloads.

Additional Build Tools 2019 or 2017 profiles are created only if MSBuild 16 or 15 is a hard POC requirement and Microsoft licensing, servicing, base-OS compatibility, and workload tests pass. MSBuild 2.0 through 14 require an explicit support decision and are not in the standard set.

```text
Harness MSBuild Plugin
-> Harness VS Build Tools 2022 Windows runtime
-> solution + targets + configuration + platform
-> MSBuild response file
-> Harness logs, artifacts, and test handoff
```

Microsoft supports Visual Studio Build Tools in Windows containers and publishes official Windows .NET SDK and .NET Framework SDK images. Full Visual Studio editions are not supported in Windows containers. Therefore `devenv.exe` tasks must be converted to Build Tools/MSBuild-compatible commands or identified as unsupported for this Kubernetes Windows target. Installer, SSDT/database, older SDK, and some C++ workloads must be tested before they are added to a Harness profile.

## What we still need to confirm

- Which project types and Visual Studio workload/component IDs block the POC?
- Which builds require MSBuild 15/16 or older versions rather than MSBuild 17?
- Which tasks call `devenv.exe` and can they be converted to MSBuild?
- What Windows LTSC baseline and container isolation mode will Kubernetes use?

## Customer position

- Harness will own the .NET and Build Tools Windows images used by the plugin.
- Modern .NET, supported .NET Framework, and MSBuild 17 have clear container implementation paths.
- Legacy MSBuild profiles require explicit support decisions.
- Full Visual Studio and unchanged `devenv.exe` execution are not supported in Windows containers.

## Sources

- [Atlassian MSBuild task](https://confluence.atlassian.com/bamboo0900/msbuild-1167721261.html)
- [Atlassian Visual Studio task](https://confluence.atlassian.com/bamboo/visual-studio-289277041.html)
- [Microsoft Build Tools in Windows containers](https://learn.microsoft.com/en-us/visualstudio/install/build-tools-container)
- [Microsoft Visual Studio container support](https://learn.microsoft.com/en-us/troubleshoot/developer/visualstudio/installation/visual-studio-unsupported-operating-systems)
- [Microsoft .NET container images](https://learn.microsoft.com/en-us/dotnet/core/docker/container-images)
