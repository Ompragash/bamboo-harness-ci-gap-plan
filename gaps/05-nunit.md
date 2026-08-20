# NUnit

## Customer need

The customer runs NUnit tests across projects associated with Visual Studio 2015, 2017, 2019, 2022, and a label “Visual Studio 2025.” Harness must execute the correct runner in Windows containers, apply assemblies/projects, categories, filters, and arguments, fail correctly, and publish accurate test results.

## How Bamboo handles it

Bamboo selects a long-lived Windows agent with the configured NUnit executable and .NET/Visual Studio runtime already installed. The NUnit task constructs the runner command, writes NUnit XML, and parses the results into Bamboo.

```text
Bamboo requires NUnit + .NET capabilities
-> matching Windows agent is selected
-> NUnit task executes tests and parses results
```

## Harness implementation

Recommendation: execute NUnit in a native Run step using an explicit Harness-maintained .NET test image. Do not build an NUnit execution plugin.

Modern projects use the matching `harness/windows-dotnet-sdk:<major>-<ltsc>` image with the repository's NUnit adapter packages. Legacy console-runner projects use a bounded `harness/windows-dotnet-test:nunit3-<vs-profile>-<ltsc>` tag derived from the approved Build Tools profile. That image contains the supported NUnit console runner and a pinned, non-destructive NUnit/TRX-to-JUnit conversion tool.

```text
Harness Run step
-> explicit .NET SDK or NUnit/Build Tools test image
-> dotnet test or nunit3-console.exe
-> NUnit XML/TRX converted to JUnit when required
-> Harness Report Paths publish the JUnit result
```

Harness currently requires JUnit XML for the Tests tab. Conversion tooling therefore belongs in the Harness image rather than being downloaded during each test run. The Run template standardizes runner arguments, filters, report location, conversion, and failure behavior.

`drone-nunit` is not an NUnit runner. It is a Linux-only, in-place NUnit-to-JUnit postprocessor with no supported Windows release and known result-handling limitations. It can inform converter tests, but it is not the production execution image and does not justify an NUnit Plugin step.

Windows C# Test Intelligence is not committed by this plan because current support for the customer's Windows/framework combinations is not established. The Run path provides full execution and result visibility without that claim.

## What we still need to confirm

- Which NUnit runner, adapter, and .NET framework versions are active?
- Which assemblies, categories, filters, and result formats are configured?
- Is Windows C# Test Intelligence a POC requirement?
- What does the label “Visual Studio 2025” mean?

## Customer position

- NUnit uses a native Run step and explicit Harness-owned .NET test image.
- Test execution and JUnit conversion are both prepared in that image.
- No NUnit execution plugin is required.
- Windows Test Intelligence remains an explicit support question.

## Sources

- [Bamboo NUnit Runner](https://confluence.atlassian.com/display/BAMBOO/NUnit%2BRunner)
- [`drone-nunit`](https://github.com/harness-community/drone-nunit)
- [Harness .NET test-report guidance](https://developer.harness.io/docs/continuous-integration/development-guides/ci-dotnet/)
- [Harness test report reference](https://developer.harness.io/docs/continuous-integration/use-ci/run-tests/test-report-ref/)
