# NUnit

## Customer need

The customer needs NUnit execution and build-visible results across projects associated with Visual Studio 2015, 2017, 2019, 2022, and a customer label “Visual Studio 2025.” The runner major, .NET Framework versus modern .NET split, assemblies/projects, categories, and expected report format are unknown.

The required outcome has two parts: execute the correct NUnit runner with the right filters and fail semantics, then show accurate results in Harness. Test Intelligence is a separate optional outcome and is not currently documented as supported for a specific Windows C# version.

## What Bamboo provides

Bamboo's NUnit Runner selects a configured NUnit executable capability, accepts test assemblies/projects, result filename, include/exclude categories, command-line options, and environment variables, runs NUnit, and parses results. Bamboo's useful abstraction combines runner execution and visible test results.

```text
Bamboo NUnit task
-> selected NUnit executable
-> assemblies, categories, options
-> runner exit plus NUnit XML
-> Bamboo test results
```

Atlassian has historically forced NUnit 3 output into NUnit 2 compatibility format in this task, which makes the customer's configured runner and result files important migration evidence.

## Harness today

Harness can execute `dotnet test` or `nunit3-console.exe` in a Run or Test step and ingest compatible reports. Harness documentation includes a Windows NUnit-to-JUnit transform example. Current Test Intelligence documentation lists Windows C# supported versions as TBD, so .NET 6+ must not be treated as proof of Windows TI support.

`harness-community/drone-nunit` was reviewed at `479806210a6e95b96bc24eefb9f3d41dd953ab4c`. It does not execute NUnit or publish directly to Harness. It finds existing NUnit XML, reads NUnit 3 `<test-run failed>`, optionally fails, transforms NUnit 2/3 to JUnit through XSLT, and overwrites the source file. It is Linux amd64 only, uses CGO with libxml2/libxslt, has no tags/releases, and its release pipeline references missing files and the wrong default branch. Static review also found that NUnit 2 root failures are not counted for gating and parse errors can be deferred. Its checked-in empty fixture is malformed. `xsltproc` successfully transformed the two valid fixtures, but Go tests/builds could not be run because Go is not installed in the research environment.

## Gap

The gap is qualification of the actual runner/runtime and a safe report path on Windows. `drone-nunit` cannot replace Bamboo's runner and is not Windows-ready. Porting it unchanged would retain destructive output, CGO dependencies, NUnit 2 gating defects, and stale release automation.

## Recommended approach

Recommendation: execute NUnit natively in Harness, publish a runner-produced supported report when possible, and use a side-by-side transform only for legacy formats.

For modern projects, qualify the Harness Test step or a governed Run step with `dotnet test`. For legacy assemblies, use the selected NUnit console executable in the .NET workload image. Preserve the original NUnit XML and write JUnit to a separate path before native ingestion.

Do not extend `drone-nunit` for Windows as the POC default. Reuse its XSLT only after license/security review and verified fixtures if it matches the required legacy format. If NUnit, MSTest, and Cucumber all need normalization, prefer one shared cross-platform normalizer with format-specific adapters and no CGO.

## POC experience

The customer configures a native Test or governed Run template with the runner, project/assembly paths, categories/filters, options, result destination, and report path.

Proposed template inputs, not final Harness YAML:

```yaml
runner: nunit3-console
assemblies: [tests/Product.Tests.dll]
includeCategories: [Smoke]
nunitReport: TestResults/nunit.xml
junitReport: TestResults/junit.xml
failIfNoResults: true
failedTestsFailBuild: true
```

The transform is omitted when the selected runner can produce a directly supported result. Test Intelligence is disabled unless its current Windows compatibility and an E2E proof support the exact project.

## Productized direction

Prefer native report-format support. If a normalizer is required, create a signed Windows/Linux utility that reads source reports without mutation, writes a separate JUnit file, emits passed/failed/skipped totals, distinguishes parse failure from test failure, supports explicit empty-result policy, and has fixtures for NUnit 2/3, MSTest/TRX, and selected Cucumber formats. It does not invoke test runners or publish to qTest.

The existing repository can be superseded or retired after the shared design is approved. Source existence is not a support commitment.

## Discovery required

- Which NUnit major, executable, framework, assemblies/projects, categories, and options are active?
- Does Bamboo currently emit NUnit 2 compatibility XML, NUnit 3 XML, TRX, or JUnit?
- Are attachments, custom adapters, fail-if-no-results, or result conversion required?
- Is Windows C# Test Intelligence a POC requirement or is native result visibility sufficient?

## Validation

Run representative modern and legacy suites on the target Windows environment. Verify passed, failed, skipped, filtered, and empty cases; correct process/task failure; NUnit 2 and 3 fixture handling if both are active; paths with spaces; adapter discovery; preserved source XML; accurate Tests-tab counts; malformed input; cancellation; and secret masking. Compare the same suite and report counts with Bamboo.

## Effort and ownership

- POC: qualification only when the runner already exists; 1 to 2 engineering weeks if a shared legacy image/template/transform is required.
- Product normalizer: 2 to 4 engineering weeks only after multi-format need is confirmed.
- Likely ownership: CI; Platform participates in binary/image release.

## What we can tell the customer

- Harness will run the real NUnit runner and display its results; the community plugin does not execute tests.
- The current community plugin is Linux-only and needs more than a Windows Dockerfile, so it will not be presented as the POC solution.
- Legacy report conversion is available as a bounded path while native support is preferred.
- Windows C# Test Intelligence remains unconfirmed until the exact project passes supported-matrix and E2E checks.

## Sources

- [Atlassian NUnit Runner](https://confluence.atlassian.com/display/BAMBOO/NUnit%2BRunner)
- [Atlassian BAM-18336 NUnit 3 compatibility behavior](https://jira.atlassian.com/browse/BAM-18336)
- [`drone-nunit` at `479806210a6e95b96bc24eefb9f3d41dd953ab4c`](https://github.com/harness-community/drone-nunit/tree/479806210a6e95b96bc24eefb9f3d41dd953ab4c)
- Harness local evidence: `developer-hub` `1c7c98f1d76bb7b8330d6ffba96f984878a32748`, `run-tests-in-ci.md`, `tests-v2.md`, and `test-report-ref.md`.
