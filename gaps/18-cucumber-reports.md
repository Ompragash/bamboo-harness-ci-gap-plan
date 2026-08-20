# Cucumber reports

| Field | Value |
| --- | --- |
| Bamboo plugin key | cucumber-bamboo-plugin |
| Provider | Third party |
| Customer version(s) | Not provided |
| Harness CSV status | No |
| Scope | CI, native reporting or existing plugin qualification/extension |
| Recommended Harness approach | Prefer native JUnit ingestion; repair drone-cucumber only for confirmed threshold/output needs |
| Solution type | I. Discovery required before selecting implementation |
| Discovery required | Yes |
| Planning confidence | Low |

## 1. What this Bamboo task does

The plugin parses Cucumber results and turns scenarios into build-visible test data. Marketplace behavior also includes fail-on-no-tests, tag-based rules, thresholds, a Bamboo report tab, and optional Jira issue actions.

It does not execute Cucumber tests. Its value is converting and acting on structured results.

## 2. How it works in Bamboo

Test runner emits Cucumber result files → Bamboo Cucumber task parses scenarios/steps → thresholds and optional Jira actions → Bamboo report and task result.

Material inputs include result format/version, include/exclude globs, thresholds, tags, fail-on-no-tests, and optional Jira configuration.

## 3. How the customer uses it

Confirmed customer usage: the plugin is in the inventory. No version, JSON/JUnit format, globs, thresholds, tags, report-tab need, or Jira/qTest behavior is supplied.

Typical plugin capability: scenario-level display and quality gates from Cucumber result files.

Customer usage context: not confirmed from the available source material.

Smallest question: Is the required outcome only scenario results in Harness, or are legacy JSON parsing, thresholds, tag rules, fail-on-no-tests, HTML/report-tab, or Jira actions used?

## 4. What Harness supports today

Harness can ingest JUnit XML from Run or Test steps and display parsed results. Cucumber runners can emit or be configured to produce JUnit, which may satisfy the core scenario-result outcome without a plugin.

An existing drone-cucumber repository has pure-Go code and an LTSC 2022 Dockerfile. At reviewed commit a39f074aa8ee6e77e9f17495ace6dc2ab45fd778, checked-in tests pass, but targeted review found material defects: exclude patterns are ignored, recursive double-star globs are not implemented, feature/scenario thresholds use failed-step counts, malformed JSON is swallowed, and defaults do not reach execution. It does not produce JUnit/HTML or implement Jira actions.

The CSV says No because exact customer behavior is unknown and the existing plugin is not release-qualified.

## 5. The actual gap

If scenario-level JUnit visibility is enough, native report ingestion already solves the outcome with configuration and qualification. If legacy Cucumber JSON thresholds and Harness outputs are required, drone-cucumber needs repair. Jira actions or a Bamboo-style report tab are separate integration/product scopes.

## 6. Recommended Harness solution

Recommendation: use the Cucumber runner's JUnit formatter and native Harness report ingestion by default; repair the existing drone-cucumber plugin only for confirmed threshold or consolidated-output requirements.

The native path lets the customer configure report paths on a Run/Test step and use the Tests tab. The plugin path exposes result globs, thresholds, tags if scoped, and deterministic outputs.

Engineering for the conditional plugin path fixes the confirmed include/exclude, recursive-glob, threshold, defaults, malformed-input, and output-failure defects; produces deterministic metrics/outputs; and adds a Windows Kubernetes smoke test. The Cucumber runner, not the plugin, remains responsible for JUnit generation. Packaging/release automation, new result formats, and Jira behavior are outside the bounded repair estimate. We should not create a second Cucumber plugin.

Result: the POC uses the smallest reliable path while preserving an extension route for required gates.

## 7. Proposed implementation shape

- Native path: language-specific Cucumber test command emits JUnit; Run/Test step report paths ingest it.
- Existing repository: harness-community/drone-cucumber at a39f074aa8ee6e77e9f17495ace6dc2ab45fd778.
- Required hardening if used: include/exclude, recursive glob, correct feature/scenario metrics, defaults, malformed/error failure, output-write failure, and deterministic step outputs.
- Format strategy: legacy JSON only if required; evaluate Cucumber Messages/NDJSON separately.
- Tests: fixtures for versions/tags/hooks/backgrounds, empty and malformed reports, Windows paths, large suites, thresholds, and outputs.
- Explicit exclusions: Jira/qTest synchronization and full Bamboo report-tab parity until separately discovered.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which Cucumber plugin/version and output format are used? | Selects native JUnit versus parser work. | Customer |
| Which globs, thresholds, tag rules, and fail-on-no-tests settings are active? | Defines plugin repair scope. | Customer |
| Are Jira actions or HTML/report-tab behavior required? | These are separate integration/UI work. | Customer / Product |
| Is qTest publishing part of this step or a separate qTest task? | Prevents duplicate result submission scope. | Customer |

## 9. Validation plan

Use representative customer results and a runnable sample. Verify scenario counts/names, passed/failed/skipped, tags, empty and malformed input, include/exclude globs, recursive paths, thresholds, large files, JUnit display in Harness, Windows Kubernetes execution, outputs, and task failure. Compare the same report with Bamboo. Test Jira only if separately approved.

## 10. Dependencies and risks

- Blocking: report fixtures and exported settings are missing.
- Planning: native reporting and Bamboo plugin parity have very different scope.
- Implementation: confirmed defects and legacy JSON format.
- Long-term maintenance: Cucumber result-format evolution and absent current release automation.

## 11. Planning estimate

Discovery required before estimate. Native JUnit mapping is qualification only. Conditional drone-cucumber repair is 1 to 2 engineering weeks only for the bounded core parser/threshold defects and one Windows smoke path. Packaging/release automation, Jira, JUnit generation, or new-format support require a separate estimate.

## 12. What we can tell the customer now

- Harness can display Cucumber scenarios when the runner emits JUnit-compatible results.
- An existing Windows-oriented Cucumber plugin is available as a starting point but requires repair and qualification.
- We will not create a duplicate plugin.
- Exact report format, thresholds, tags, and Jira behavior are needed before selecting the path.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 32.

### Bamboo/vendor

- [Cucumber for Bamboo Marketplace listing](https://marketplace.atlassian.com/apps/1214740/cucumber-for-bamboo)
- [Cucumber continuous integration guidance](https://cucumber.io/docs/guides/continuous-integration/)
- [Cucumber Messages](https://github.com/cucumber/messages)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/run-tests/test-report-ref.md
- [drone-cucumber at a39f074aa8ee6e77e9f17495ace6dc2ab45fd778](https://github.com/harness-community/drone-cucumber/tree/a39f074aa8ee6e77e9f17495ace6dc2ab45fd778)
- Repository audit at a39f074aa8ee6e77e9f17495ace6dc2ab45fd778: go test ./..., go test -race ./..., go vet ./..., and a Windows AMD64 cross-build passed; temporary targeted tests reproduced four parser/failure-semantics defects and were removed after the audit.
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/PluginStepInfo.java

Confidence: Low.
