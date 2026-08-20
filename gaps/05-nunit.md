# NUnit

## Customer need

The customer runs NUnit tests for projects associated with Visual Studio 2015, 2017, 2019, 2022, and a label “Visual Studio 2025.” Harness must execute the correct NUnit runner in a Windows container, apply assemblies/projects, categories, filters, and arguments, fail correctly, and publish accurate results.

Test execution and result conversion are separate responsibilities, but the customer should see one structured NUnit task.

## How Bamboo handles it

Bamboo selects a Windows agent with the configured NUnit executable and required .NET/Visual Studio runtime already installed. The NUnit task receives test assemblies or projects, result filename, include/exclude categories, command-line options, and environment variables.

The task starts NUnit and parses the result into Bamboo test reporting. Bamboo has historically asked NUnit 3 to produce NUnit 2-compatible output, so the exact customer report format matters.

```text
Bamboo selects agent with NUnit + required .NET runtime
-> NUnit task runs assemblies with filters/options
-> NUnit XML is parsed
-> test result and task status return to Bamboo
```

## Harness implementation

Recommendation: build a Harness NUnit plugin on the same Harness-owned .NET/Visual Studio runtime family used by MSBuild and MSTest.

The plugin selects either the modern .NET SDK profile or a supported Visual Studio Build Tools/.NET Framework profile. It invokes `dotnet test` or the supported NUnit console runner, writes the original NUnit report, converts only when Harness needs JUnit, and publishes the result through Harness test reporting.

```text
Harness NUnit Plugin
-> Harness .NET/VS test Windows runtime
-> NUnit runner
-> original NUnit XML
-> optional non-destructive JUnit conversion
-> Harness test results
```

`drone-nunit` does not execute tests. It is a Linux-only postprocessor that converts existing NUnit XML to JUnit in place and can fail on detected NUnit 3 failures. It uses CGO/libxml2/libxslt, has no supported Windows release, overwrites the source report, and does not correctly apply its root-failure gate to NUnit 2 reports.

Harness should reuse only verified transformation behavior from that repository. The supported plugin needs a Windows-native, non-destructive conversion component, correct NUnit 2/3 failure semantics, structured counts, malformed/no-result handling, tests, signed images, and release ownership.

## What we still need to confirm

- Which NUnit runner versions and .NET frameworks are required?
- Which assemblies/projects, categories, filters, and result formats are configured?
- Is Windows C# Test Intelligence a POC requirement?
- What does the label “Visual Studio 2025” mean?

## Customer position

- Harness will provide one structured NUnit execution and reporting flow on Windows Kubernetes.
- NUnit will reuse Harness-maintained .NET/Visual Studio runtime images.
- The existing community plugin is useful only as result-conversion reference code, not as the runner.
- Windows C# Test Intelligence still requires an explicit supported-version decision.

## Sources

- [Atlassian NUnit Runner](https://confluence.atlassian.com/display/BAMBOO/NUnit%2BRunner)
- [Atlassian NUnit 3 compatibility behavior](https://jira.atlassian.com/browse/BAM-18336)
- [`drone-nunit`](https://github.com/harness-community/drone-nunit)
- [Harness test report reference](https://developer.harness.io/docs/continuous-integration/use-ci/run-tests/test-report-ref/)
