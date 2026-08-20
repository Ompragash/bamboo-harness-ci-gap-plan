# MSTest and VSTest

## Customer need

The customer runs Microsoft tests for projects associated with Visual Studio 2015, 2017, 2019, 2022, and a label “Visual Studio 2025.” Harness must support the required `dotnet test`, `vstest.console.exe`, or legacy MSTest mode, including adapters, filters, `.runsettings`, result paths, and failure behavior.

The task runs in a Windows container and should reuse the same Harness-owned .NET/Visual Studio foundation as MSBuild and NUnit.

## How Bamboo handles it

Bamboo selects a long-lived Windows agent with the configured Microsoft test executable and Visual Studio/.NET runtime already installed. The MSTest task supplies test containers and runner options, invokes the executable, and parses the results into Bamboo.

```text
Bamboo selects agent with Microsoft test runner + adapters
-> task runs assemblies/projects with settings and filters
-> TRX/result file is parsed
-> test result and task status return to Bamboo
```

## Harness implementation

Recommendation: build a Harness MSTest execution wrapper on the Harness-owned .NET/Visual Studio runtime family.

The wrapper chooses the modern .NET SDK profile for `dotnet test` or a supported Visual Studio Build Tools test profile for VSTest and full .NET Framework. The runtime image contains the supported test platform and standard adapters before the container starts. Additional adapters or data collectors are packaged only when the selected customer projects require them.

```text
Harness MSTest Wrapper
-> Harness .NET SDK or VS Build Tools test runtime
-> dotnet test or vstest.console.exe
-> TRX
-> Harness test results
```

Proposed inputs: runtime profile, project/assembly paths, adapter path, `.runsettings`, filter, configuration, results directory, report paths, and additional runner arguments. Harness manages logs, secrets, timeout, failure strategy, and test result publication.

No separate MSTest runtime image is required. The MSBuild, NUnit, and MSTest components share the same .NET SDK and Build Tools profiles. Legacy `mstest.exe` or old adapters are added only after an explicit Harness support decision.

Current Harness documentation does not identify supported Windows C# Test Intelligence versions. The POC can provide complete execution and result visibility without claiming TI until the exact framework is supported and tested.

## What we still need to confirm

- Which runner executable, adapter version, and framework are active?
- Which `.runsettings`, filters, coverage collectors, and deployment items are required?
- Is legacy `mstest.exe` required, or can builds use VSTest/`dotnet test`?
- Is Windows C# Test Intelligence required for the POC?

## Customer position

- Harness will provide a structured Microsoft test flow on Windows Kubernetes.
- MSTest reuses Harness-maintained .NET/Visual Studio runtime images.
- Modern `dotnet test` and supported VSTest modes have clear implementation paths.
- Legacy runners and Windows Test Intelligence require explicit support decisions.

## Sources

- [Atlassian MSTest Runner](https://confluence.atlassian.com/display/BAMBOO1020/MSTest%2BRunner)
- [Microsoft .NET container images](https://learn.microsoft.com/en-us/dotnet/core/docker/container-images)
- [Microsoft Build Tools in Windows containers](https://learn.microsoft.com/en-us/visualstudio/install/build-tools-container)
- [Harness Test step](https://developer.harness.io/docs/continuous-integration/use-ci/run-tests/tests-v2/)
