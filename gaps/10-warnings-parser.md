# Warnings parser

| Field | Value |
| --- | --- |
| Bamboo plugin key | atlassian-bamboo-warnings:task.warnings.parser |
| Provider | Bamboo native |
| Customer version(s) | Not provided |
| Harness CSV status | No |
| Scope | CI, discovery and conditional parser integration |
| Recommended Harness approach | Parse known formats in a template and publish a Harness summary; evaluate platform work only if file-level annotations are required |
| Solution type | I. Discovery required before selecting implementation |
| Discovery required | Yes |
| Planning confidence | Low |

## 1. What this Bamboo task does

The task scans compiler or analysis output, recognizes warnings, and presents structured warning counts and details in Bamboo. It can turn otherwise unstructured logs into a quality signal and may fail a build based on configured criteria.

The value is result parsing and presentation, not command execution.

## 2. How it works in Bamboo

Build tool writes console or file output → Bamboo warnings parser applies a selected format → warning records and counts → Bamboo build summary and optional failure behavior.

Material inputs are parser type, input files/logs, thresholds, and inclusion rules. File, line, severity, and message fields matter if the customer relies on navigation rather than only counts.

## 3. How the customer uses it

Confirmed customer usage: the task is in the inventory. No parser formats, input paths, thresholds, or required UI behavior are supplied.

Typical plugin capability: parse compiler warnings, show them in a build report, and enforce warning thresholds.

Customer usage context: not confirmed from the available source material.

Smallest question: Which parser formats and thresholds are configured, and is file/line UI navigation required or are summary counts and build failure enough?

## 4. What Harness supports today

Harness retains execution logs and can upload generated reports/artifacts. Pipeline Annotations allow a step to publish Markdown summaries through hcli on Kubernetes infrastructure, with documented size/count limits. A Run template can execute a parser, fail on a threshold, emit outputs, and publish a summary.

No verified Harness contract was found for ingesting arbitrary compiler warnings as native file-and-line annotations across the CI UI. The CSV says No because Bamboo's structured warning report is not a confirmed native equivalent.

## 5. The actual gap

Summary counts and thresholds can be covered with a parser plus native annotations. Exact Bamboo parity may require a structured warning ingestion model containing file, line, column, severity, message, deduplication, and trend behavior.

The implementation choice depends on the outcome the customer actually uses.

## 6. Recommended Harness solution

Recommendation: first qualify a reusable parser template for the customer's known formats, producing step outputs, a Markdown annotation, and threshold failure.

The user selects parser preset, input paths, include/exclude rules, and thresholds. Harness supplies the Windows image, secrets, logs, outputs, annotation, timeout, failure strategy, and template versioning.

Engineering work for the template is format parsing, Windows path handling, deterministic summaries, tests, and hcli integration. If clickable file/line annotations or historical trend are P0 requirements, evaluate a product enhancement instead. We should not commit to a generic warnings plugin before the UI/API contract and parser formats are known.

Result: the POC can demonstrate visible warning summaries and gates, with any deeper platform gap explicitly separated.

## 7. Proposed implementation shape

- Initial parser: only customer-used MSBuild/MSVC/Java or regex formats.
- Step: Run or Plugin step on Windows Kubernetes, with input file globs and encoding.
- Outputs: total, new if a baseline exists, severity counts, and report artifact path.
- UI: hcli Markdown annotation with bounded top findings and link to full artifact.
- Failure: configurable count/severity thresholds and malformed-input failure.
- Conditional platform work: structured warning schema, ingestion endpoint, YAML setting, Tests-like UI, file navigation, and retention.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which parser presets, encodings, and file globs are configured? | Defines the actual parser scope. | Customer |
| Which thresholds change build status? | Defines required result semantics. | Customer |
| Are inline file/line links or historical trends required? | May turn a template into platform work. | Customer / Product |
| Is Kubernetes infrastructure used for every relevant pipeline? | Current annotation documentation is infrastructure-limited. | Customer / Engineering |

## 9. Validation plan

Use captured non-confidential outputs from each active compiler. Verify Windows absolute and relative paths, spaces, Unicode, malformed input, duplicate warnings, severity mapping, thresholds, empty files, large reports, annotation limits, artifact link, and failure behavior. Compare counts with the corresponding Bamboo build.

## 10. Dependencies and risks

- Blocking: parser configurations and sample outputs are missing.
- Planning: summary-only and clickable file/line experiences are different scopes.
- Implementation: format drift, Windows path parsing, and annotation limits.
- Long-term maintenance: every parser preset becomes a compatibility obligation.

## 11. Planning estimate

Discovery required before estimate. A bounded template for one to three known formats may fit within 1 to 2 engineering weeks. Native structured warning ingestion would be a separate product workstream.

## 12. What we can tell the customer now

- Harness provides logs, artifacts, outputs, and Markdown annotation primitives that can host this workflow; no qualified warning parser/template exists today.
- We need the exact warning formats and required UI outcome before choosing template versus platform work.
- We are not assuming that every Bamboo parser preset must be reproduced.
- File-and-line native annotation parity is not yet confirmed.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 16.

### Bamboo/vendor

- [Atlassian: Configuring the build warnings parser task](https://confluence.atlassian.com/bamboo0800/configuring-build-warnings-parser-task-1077778946.html)
- [GitHub Actions: Workflow commands and warning annotations](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-commands)
- [Buildkite: Annotations](https://buildkite.com/docs/pipelines/configure/annotations)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/platform/pipelines/harness-annotations.md
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/platform/templates/run-step-template-quickstart.md
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/PluginStepInfo.java

Confidence: Low.
