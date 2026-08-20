# MSTest runner

| Field | Value |
| --- | --- |
| Bamboo plugin key | plugin.dotnet:mstest |
| Provider | Bamboo native .NET plugin |
| Customer version(s) | MSTest through Visual Studio 2015, 2017, 2019, 2022, and customer-provided label “Visual Studio 2025”; label requires confirmation |
| Harness CSV status | Partial |
| Scope | CI, native Test/TI and legacy-mode discovery |
| Recommended Harness approach | Qualify the native Test/reporting path on Windows; use a governed VSTest/report fallback for unqualified modes |
| Solution type | B. Existing native capability plus qualification |
| Discovery required | Yes |
| Planning confidence | Medium |

## 1. What this Bamboo task does

The task invokes Microsoft's test runner for selected assemblies or test containers and publishes the resulting test status in Bamboo. It provides runner/toolchain selection, filters, result collection, and failure behavior around MSTest or VSTest.

## 2. How it works in Bamboo

Bamboo job → MSTest task → selected Visual Studio test runner → test assemblies and filters → TRX/result file → Bamboo result parser and task status.

Compatibility depends on the project runtime, VSTest version, test adapter, targeting pack, and result format.

## 3. How the customer uses it

Confirmed customer usage: the inventory associates MSTest with Visual Studio 2015 through its label “Visual Studio 2025.” Confirm whether that final label means a 2025 runner image/servicing date, Visual Studio 2022 17.x, or another toolchain.

Typical plugin capability: run MSTest/VSTest assemblies and publish their results.

Customer usage context: not confirmed from the available source material.

Smallest question: Which active plans use dotnet test, vstest.console.exe, or legacy mstest.exe, and what adapters or test settings files do they require?

## 4. What Harness supports today

Harness provides a native Test step with reports, outputs, environment, shell, splitting, and Test Intelligence integration. However, the current C# compatibility table lists Windows supported versions as TBD and .NET Framework as TBD. The .NET Core 6+ entry is an implementation minimum, not proof of Windows Test Intelligence support.

A governed Run step can invoke legacy VSTest and publish compatible reports. Harness documentation describes TRX handling for C# and converter-based JUnit ingestion.

The CSV is Partial because governed execution and reporting exist, but Windows Test-step behavior, the customer's runner matrix, and TI compatibility are not proven.

## 5. The actual gap

The missing work is qualification for the active MSTest/VSTest modes, adapters, settings files, and result conversion on Windows Kubernetes. It is not a need to recreate Microsoft's test runner.

## 6. Recommended Harness solution

Recommendation: first qualify the native Test step with a representative modern Windows MSTest project; use it only when that proof passes, and otherwise use a reusable Run template with VSTest/report ingestion. Apply the same fallback to confirmed legacy modes.

Harness manages the Windows image, credentials, environment, logs, outputs, reports, timeout, failure strategy, and optional TI/splitting where supported. Engineering qualifies representative projects and packages only required adapters or conversion tools.

We should not build an MSTest plugin whose only purpose is calling a Microsoft runner. Result: test results remain native to Harness while legacy constraints are explicit.

## 7. Proposed implementation shape

- Candidate modern path: pinned .NET SDK image, native Test step, dotnet test, TRX configuration; TI only after Windows qualification.
- Legacy path: shared Build Tools image, vstest.console.exe through a Run template, deterministic report conversion/ingestion.
- Inputs: project or assembly glob, adapter path, test settings, filters, configuration, results directory, reports, timeout, and failure strategy.
- Qualification: one sample for each active runner family, including data-driven, skipped, failed, and filtered tests.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which runner executable and MSTest adapter versions are active? | Defines the execution and report path. | Customer |
| Which workloads are modern .NET versus full .NET Framework? | Determines native Test/TI eligibility. | Customer |
| Are .runsettings, deployment items, code coverage, or custom adapters used? | May require extra image contents and outputs. | Customer |
| Is TI required in the POC? | Sets a stricter supported-framework acceptance criterion. | Customer / Product |

## 9. Validation plan

Execute representative modern and legacy suites on the target Windows Kubernetes node. Confirm passed, failed, skipped, filtered, and data-driven result counts; adapter discovery; .runsettings behavior; task failure; Tests-tab visibility; paths with spaces; cancellation; and secret masking. Treat Windows C# TI as unconfirmed until the compatibility owner and an E2E proof establish support.

## 10. Dependencies and risks

- Blocking: missing sample projects and runner details.
- Planning: Visual Studio version alone does not identify test runtime.
- Implementation: custom adapters, data collectors, and result conversion.
- Long-term maintenance: legacy test platforms and targeting packs.

## 11. Planning estimate

Qualification only for the first modern Windows Test-step proof. A shared legacy runner/image/template path may take 1 to 2 engineering weeks and must be counted once with MSBuild and NUnit.

## 12. What we can tell the customer now

- Harness has native Test and report contracts, but current documentation leaves Windows C# supported versions TBD.
- Legacy VSTest execution can be governed and reported through Harness, but needs sample-based qualification.
- Windows C# Test Intelligence is not being claimed until the exact project passes qualification.
- The active runner, adapter, and runtime details are needed before confirmation.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 8.

### Bamboo/vendor

- [Atlassian: MSTest Runner](https://confluence.atlassian.com/display/BAMBOO1020/MSTest%2BRunner)
- [Atlassian: Configuring a test task](https://confluence.atlassian.com/bamboo1200/configuring-a-test-task-1680480840.html)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/run-tests/tests-v2.md
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/run-tests/test-report-ref.md
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/TestStepInfo.java

Confidence: Medium.
