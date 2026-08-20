# MSBuild and Visual Studio

## Customer need

The customer lists MSBuild 2.0, 3.5, 4.0, 12, 14, 15, 16, and 17. Harness must build the selected Windows solutions and projects with the necessary .NET SDKs, .NET Framework targeting packs, C++ toolsets, and other workloads on Kubernetes Windows workers.

The POC must distinguish Build Tools-compatible builds from tasks that require full Visual Studio or `devenv.exe`.

## How Bamboo handles it

Bamboo administrators install MSBuild or Visual Studio on long-lived Windows agents and register executable capabilities. The task adds the selected executable requirement, so Bamboo schedules the job only on a matching agent.

The MSBuild task supplies a project, arguments, environment, and working directory and can invoke MSBuild through a response file. The Visual Studio task can invoke `devenv.exe` after initializing the installed Visual Studio environment.

```text
Bamboo task requires a registered MSBuild/Visual Studio executable
-> Bamboo selects the matching Windows agent
-> task constructs the build invocation
-> installed toolchain executes
```

## Harness implementation

Recommendation: use explicit Harness-maintained .NET or Visual Studio Build Tools images through native Run steps. Do not build an MSBuild plugin for the POC.

The image is the substantive capability. Proposed families are:

- `harness/windows-dotnet-sdk:<major>-<ltsc>` for modern SDK-style projects;
- `harness/windows-dotnet-framework:<target>-<ltsc>` for supported full-framework builds;
- `harness/windows-vs-buildtools:2022-<workload>-<ltsc>` for MSBuild 17 and confirmed Build Tools workloads.

The pipeline references the required tag explicitly. A Run Step Template standardizes solution/project, targets, configuration, platform, properties, response file, working directory, report paths, and artifacts.

```text
Harness Run step
-> explicit VS Build Tools 2022 workload image
-> MSBuild.exe solution.sln /t:Build /p:Configuration=Release
-> Harness logs, status, reports, and artifacts
```

Visual Studio Build Tools, SDKs, and targeting packs are installed when Harness builds the image, never during each pipeline run. Workload-specific tags are preferable to one enormous image containing every SDK and toolset. Microsoft does not publish one stable size for all Build Tools images because installed workload/component selections determine the footprint; Harness must measure the confirmed profiles during the image POC.

Build Tools 2019 or 2017 tags are added only for hard MSBuild 16 or 15 blockers that pass licensing, servicing, Windows-base, and workload tests. MSBuild 2 through 14 require explicit support decisions. Full Visual Studio is not supported in Windows containers, so unchanged `devenv.exe` tasks must be converted to Build Tools/MSBuild commands or marked unsupported for this target.

## What we still need to confirm

- Which project types and Visual Studio workload/component IDs block the POC?
- Which builds require MSBuild 15/16 or older rather than MSBuild 17?
- Which tasks use `devenv.exe`, and can they be converted to MSBuild?
- What Windows LTSC and container isolation mode will Kubernetes use?

## Customer position

- MSBuild uses a native Run step with an explicit Harness-owned toolchain image.
- Heavy Build Tools workloads are preinstalled in bounded workload tags.
- A dedicated MSBuild plugin does not add enough value for the POC.
- Full Visual Studio and unchanged `devenv.exe` are unsupported in Windows containers.

## Sources

- [Bamboo MSBuild task](https://confluence.atlassian.com/bamboo0900/msbuild-1167721261.html)
- [Bamboo Visual Studio task](https://confluence.atlassian.com/bamboo/visual-studio-289277041.html)
- [Harness Run step](https://developer.harness.io/docs/continuous-integration/use-ci/run-step-settings/)
- [Microsoft Build Tools in Windows containers](https://learn.microsoft.com/en-us/visualstudio/install/build-tools-container)
- [Microsoft Visual Studio container support](https://learn.microsoft.com/en-us/troubleshoot/developer/visualstudio/installation/visual-studio-unsupported-operating-systems)
