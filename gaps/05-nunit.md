# NUnit

| Field | Value |
| --- | --- |
| Bamboo plugin key | plugin.dotnet:nunit |
| Provider | Bamboo native .NET plugin |
| Customer version(s) | NUnit through Visual Studio 2015, 2017, 2019, 2022, and customer-provided label “Visual Studio 2025”; label requires confirmation |
| Harness CSV status | Partial |
| Scope | CI, native Test/TI and legacy-mode discovery |
| Recommended Harness approach | Qualify the native Test/reporting path on Windows; use a governed Run/report fallback for unqualified modes |
| Solution type | B. Existing native capability plus qualification |
| Discovery required | Yes |
| Planning confidence | Medium |

## 1. What this Bamboo task does

The task invokes NUnit tests, supplies assemblies and runner options, interprets the process result, and makes test results visible in Bamboo. Teams use it when tests are not already run as part of a Maven-like build lifecycle.

## 2. How it works in Bamboo

Bamboo job → NUnit task → selected NUnit or Visual Studio toolchain → test assemblies → NUnit result XML → Bamboo test result parser and task outcome.

The runner version, .NET runtime, assembly list, filters, and result format determine compatibility. Bamboo's useful addition is configured execution and visible test results.

## 3. How the customer uses it

Confirmed customer usage: the inventory associates NUnit with Visual Studio toolchains from 2015 through its label “Visual Studio 2025.” Confirm whether that final label means a 2025 runner image/servicing date, Visual Studio 2022 17.x, or another toolchain.

Typical plugin capability: run NUnit console or dotnet test against one or more test assemblies and publish results.

Customer usage context: not confirmed from the available source material.

Smallest question: Are the active tests modern .NET projects run by dotnet test, or legacy .NET Framework assemblies run by NUnit Console?

## 4. What Harness supports today

Harness has a native Test step with reports, outputs, environment, image, shell, test splitting, and Test Intelligence integration. However, the current C# compatibility table lists Windows supported versions as TBD and .NET Framework as TBD. The .NET Core 6+ entry is an implementation minimum, not proof of Windows Test Intelligence support. Run steps can collect JUnit-compatible reports, and the Test step documents TRX as the default C# report path.

The docs describe TRX handling in the Test step and JUnit-compatible ingestion/conversion patterns. This provides a candidate modern dotnet test path, but Windows Test-step behavior, legacy NUnit console, and full-framework modes require qualification.

The CSV is Partial because governed execution and report ingestion exist, while Windows Test-step behavior, every runner mode, and TI compatibility remain unqualified.

## 5. The actual gap

Harness can host NUnit execution and ingest compatible results. The gap is proof that each active runner/runtime combination works on the customer's Windows Kubernetes image, plus a conversion path when the runner does not produce a directly supported report.

Test Intelligence should be offered only for the documented supported combination, not implied for every Visual Studio or .NET Framework version.

## 6. Recommended Harness solution

Recommendation: first qualify the native Test step with a representative modern Windows NUnit project; use it only when that proof passes, and otherwise use a reusable Run template plus report conversion/ingestion. Apply the same fallback to confirmed legacy console workloads.

The customer configures the test project or assemblies, filters, report path, image, and secrets. Harness manages execution, selection where supported, splitting, reports, logs, timeout, failure strategy, and Tests-tab visibility.

Engineering work is qualification against representative projects and, if required, a pinned report converter in the shared .NET image. We should not build an NUnit plugin unless discovery identifies structured behavior beyond invoking the runner and ingesting its reports.

Result: native test visibility for all qualified modes, with Test Intelligence limited to supported combinations.

## 7. Proposed implementation shape

- Candidate modern path: native Test step, pinned .NET SDK image, dotnet test, TRX/report configuration; Test Intelligence and splitting only after Windows qualification.
- Legacy path: Windows Build Tools/.NET Framework image, NUnit Console through a Run template, deterministic XML conversion if needed, and report paths.
- Template inputs: project/assemblies, filters, configuration, runtime, results path, environment, secrets, timeout, and failure strategy.
- Qualification: one project per active runner family, successful and failing tests, skipped tests, attachments if used, paths with spaces, and large suites.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which NUnit major versions and runners are used? | Defines command and report compatibility. | Customer |
| Which tests are .NET Core 6+ versus full .NET Framework? | Determines native Test/TI eligibility. | Customer |
| Are filters, attachments, categories, or custom adapters required? | Defines acceptance criteria and image contents. | Customer |
| Is Test Intelligence required for the POC or only result publishing? | Changes qualification scope. | Customer / Product |

## 9. Validation plan

Run representative modern and legacy suites on the target Windows Kubernetes environment. Verify passed, failed, skipped, and filtered cases; correct task failure; report counts and names in the Tests tab; paths with spaces; proxy/CA for private packages; adapter discovery; cancellation; and secret masking. Treat Windows C# TI as unconfirmed until the compatibility owner and an E2E proof establish support, then verify selected tests and selection reasons.

## 10. Dependencies and risks

- Blocking: no representative modern and legacy test projects.
- Planning: Visual Studio version does not identify NUnit runner/runtime.
- Implementation: report conversion and custom adapters.
- Long-term maintenance: old full-framework runners and targeting packs.

## 11. Planning estimate

Qualification only for the first modern Windows Test-step proof. Use 1 to 2 engineering weeks only if a shared legacy runner image, converter, and template must be added; that work is shared with MSTest and MSBuild rather than counted per row.

## 12. What we can tell the customer now

- Harness has native Test and report contracts, but current documentation leaves Windows C# supported versions TBD.
- Legacy NUnit console modes can use governed Windows execution and report ingestion, but require qualification.
- Windows C# Test Intelligence must be confirmed against the exact runtime and framework before it is presented as supported.
- We need one modern and one legacy sample to define the POC path.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 7.

### Bamboo/vendor

- [Atlassian: NUnit Runner](https://confluence.atlassian.com/display/BAMBOO/NUnit%2BRunner)
- [Atlassian: Configuring a test task](https://confluence.atlassian.com/bamboo1200/configuring-a-test-task-1680480840.html)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/run-tests/tests-v2.md
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/run-tests/test-report-ref.md
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/TestStepInfo.java

Confidence: Medium.
