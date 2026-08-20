# MSTest and VSTest

## Customer need

The customer needs Microsoft test execution and visible results for projects associated with Visual Studio 2015, 2017, 2019, 2022, and a label “Visual Studio 2025.” The inventory does not identify `dotnet test`, `vstest.console.exe`, legacy `mstest.exe`, adapter versions, `.runsettings`, coverage collectors, or full-framework requirements.

The customer outcome is correct execution, adapters, filters/settings, task failure, and report visibility. Test Intelligence is optional and must be qualified separately.

## What Bamboo provides

Bamboo's MSTest task selects a configured test executable capability, takes test containers and result-related options, executes the Microsoft runner, and parses results into Bamboo.

```text
Bamboo MSTest task
-> selected Microsoft test executable
-> assemblies, settings, options
-> TRX/results
-> Bamboo test visibility and task outcome
```

Public Bamboo source was not available. Official task documentation defines the verified behavior; exact customer exports are needed for adapter and settings parity.

## Harness today

Harness Run and Test steps can invoke `dotnet test` or VSTest and collect reports. The Test step documents C# reporting behavior, but current compatibility tables leave Windows C# supported versions and .NET Framework support as TBD. Therefore native reporting is available as a candidate, while Windows Test Intelligence is not yet a customer commitment.

## Gap

The missing work is the exact Windows workload, runner, adapter, settings, data collector, and result-path qualification. There is no need to recreate Microsoft's runner. A plugin would only be justified if repeated result normalization is needed across NUnit/MSTest/Cucumber, not merely to invoke VSTest.

## Recommended approach

Recommendation: qualify the native Test path for modern projects and use a governed VSTest Run template plus native report ingestion for legacy or unsupported modes.

The .NET workload profile supplies the runner, targeting packs, adapters, and data collectors. Harness manages step inputs, credentials, logs, reports, outputs, timeout, failure strategy, and audit. Use a side-by-side normalizer only when the exact output is not natively supported; share it with NUnit rather than create an MSTest-only plugin.

## POC experience

Proposed template inputs, not final Harness YAML:

```yaml
runner: vstest
testContainers: [tests/Product.Tests.dll]
settingsFile: tests/ci.runsettings
filter: TestCategory=Smoke
resultsDirectory: TestResults
reportPaths: [TestResults/*.trx]
```

One modern project tests the native Test path. One full-framework project, if a blocker, tests the workload image and VSTest fallback. TI is enabled only after compatibility ownership and E2E proof.

## Productized direction

Keep runner invocation in native Test/Run steps. Maintain the selected Build Tools profiles and document adapter/report compatibility. Add TRX or legacy normalization to the shared test-result layer only when native ingestion cannot preserve required semantics.

## Discovery required

- Which runner executable, MSTest adapter version, and framework are active?
- Which `.runsettings`, deployment items, coverage/data collectors, test filters, and custom adapters are required?
- Which output format and failure/no-result behavior does Bamboo use?
- Is Windows Test Intelligence required for the POC?

## Validation

Execute representative modern and legacy suites on the target Windows environment. Verify passed, failed, skipped, filtered, and data-driven cases; adapter discovery; `.runsettings`; coverage artifact if required; accurate report counts; failure behavior; paths with spaces; cancellation; and secret masking. Compare the same project with Bamboo.

## Effort and ownership

- POC: qualification only for a modern project; legacy work is shared with MSBuild/NUnit and may take 1 to 2 engineering weeks.
- Productization: part of .NET workload maintenance; shared normalizer is 2 to 4 weeks only if selected.
- Likely ownership: CI + Platform.

## What we can tell the customer

- Harness can execute Microsoft test runners and surface reports on Windows with the right workload image.
- Modern and legacy modes will be qualified separately.
- A new MSTest invocation plugin is not planned because it would duplicate Microsoft tooling and Harness step controls.
- Windows C# Test Intelligence is not being promised until the exact mode is supported and proven.

## Sources

- [Atlassian MSTest Runner](https://confluence.atlassian.com/display/BAMBOO1020/MSTest%2BRunner)
- [Atlassian test task configuration](https://confluence.atlassian.com/bamboo1200/configuring-a-test-task-1680480840.html)
- Harness local evidence: `developer-hub` `1c7c98f1d76bb7b8330d6ffba96f984878a32748`, `run-tests-in-ci.md`, `tests-v2.md`, and `test-report-ref.md`.
