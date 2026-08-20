# MSTest and VSTest

## Customer need

The customer runs Microsoft tests across projects associated with Visual Studio 2015, 2017, 2019, 2022, and a label “Visual Studio 2025.” Harness must support the required `dotnet test`, `vstest.console.exe`, or legacy MSTest mode, including adapters, filters, `.runsettings`, result paths, and failure behavior.

## How Bamboo handles it

Bamboo selects a long-lived Windows agent with the configured Microsoft test executable, adapters, and .NET/Visual Studio runtime installed. The task constructs the command, executes the tests, and parses the result into Bamboo.

```text
Bamboo requires Microsoft test-runner capability
-> matching Windows agent is selected
-> MSTest/VSTest executes
-> result is parsed and published
```

## Harness implementation

Recommendation: execute MSTest/VSTest in a native Run step using an explicit Harness-maintained .NET or Build Tools test image. Do not build an MSTest plugin.

Modern SDK-style tests use `harness/windows-dotnet-sdk:<major>-<ltsc>`. Full-framework or VSTest workloads use a `harness/windows-vs-buildtools:2022-test-<ltsc>` tag containing the approved Test Platform, targeting packs, standard adapters, and a pinned TRX-to-JUnit converter.

```text
Harness Run step
-> explicit .NET SDK or VS Build Tools test image
-> dotnet test -l:trx or vstest.console.exe
-> pinned converter produces JUnit XML
-> Harness Report Paths publish the result
```

The converter is installed when Harness builds the image. The pipeline does not run `dotnet tool install` for every execution. A reusable Run Step Template standardizes project/assembly paths, adapter path, `.runsettings`, filters, configuration, results directory, conversion, report paths, and failure behavior.

MSTest shares its .NET and Build Tools layers with MSBuild and NUnit. Legacy `mstest.exe`, old adapters, or older Visual Studio profiles are added only after an explicit Harness support decision. Current documentation does not establish Test Intelligence support for the customer's Windows/framework combinations, so the POC does not claim TI until those combinations are qualified.

## What we still need to confirm

- Which runner executable, adapter, and framework versions are active?
- Which `.runsettings`, filters, collectors, and deployment items are required?
- Is legacy `mstest.exe` required, or can builds use VSTest/`dotnet test`?
- Is Windows C# Test Intelligence required for the POC?

## Customer position

- MSTest uses a native Run step with an explicit Harness-owned test image.
- VSTest, adapters, and report conversion are prepared in the image.
- No MSTest plugin is required.
- Legacy runners and Windows Test Intelligence require explicit support decisions.

## Sources

- [Bamboo MSTest Runner](https://confluence.atlassian.com/display/BAMBOO1020/MSTest%2BRunner)
- [Harness .NET test-report guidance](https://developer.harness.io/docs/continuous-integration/development-guides/ci-dotnet/)
- [Harness Test step](https://developer.harness.io/docs/continuous-integration/use-ci/run-tests/tests-v2/)
- [Microsoft Build Tools in Windows containers](https://learn.microsoft.com/en-us/visualstudio/install/build-tools-container)
