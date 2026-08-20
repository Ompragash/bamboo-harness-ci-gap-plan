# Cucumber results

## Customer need

The customer uses Cucumber result processing, but the plugin/version, JSON/JUnit/Messages format, globs, tags, thresholds, fail-if-no-tests behavior, report-tab need, and Jira actions are unknown.

The task does not necessarily execute tests. The desired outcome may be scenario visibility in Harness, build gating from legacy JSON, or external Jira behavior. Those are different scopes.

## What Bamboo provides

The third-party Bamboo plugin parses Cucumber results, creates build-visible scenario data, applies selected thresholds/tag rules, can fail on missing results, and may provide a report tab or Jira issue actions depending on version/configuration. Public implementation source for the exact Bamboo version was not found; marketplace/vendor behavior and exported settings are the available contract.

```text
Cucumber runner output
-> Bamboo parser
-> scenarios and thresholds
-> report/task result and optional Jira actions
```

## Harness today

Harness can ingest JUnit XML from Run or Test steps and show it in the Tests tab. Cucumber frameworks can commonly emit JUnit, so native reporting may solve the core need.

`harness-community/drone-cucumber` at `a39f074aa8ee6e77e9f17495ace6dc2ab45fd778` is a pure-Go parser with an LTSC 2022 Dockerfile. Checked-in tests and race tests passed in the prior audit, but targeted tests found that exclude patterns are ignored, `**` is not recursive, feature/scenario failure thresholds count failed steps, malformed JSON is swallowed, and defaults do not reach execution. It neither generates JUnit/HTML nor performs Jira actions. Source existence is useful, but it is not ready for a supported POC without repair.

## Gap

If JUnit scenario visibility is enough, the gap is qualification only. If legacy JSON thresholds, tag rules, or structured outputs are required, the existing plugin needs bounded repair. Jira automation and a Bamboo-style report tab are separate integration/product decisions.

## Recommended approach

Recommendation: have the Cucumber runner emit JUnit and use native Harness reporting by default; repair `drone-cucumber` only for confirmed JSON gate behavior.

Do not create a second Cucumber plugin. If repaired, fix include/exclude/recursive globs, scenario versus step metrics, defaults, malformed/no-result policy, output-write errors, deterministic outputs, Windows paths, and release automation. The plugin remains a parser/gate, not the test runner or qTest publisher.

## POC experience

Native path:

```text
language-specific Cucumber command
-> JUnit report
-> Harness report path
-> Tests tab and native failure strategy
```

Conditional proposed plugin inputs, not final Harness YAML:

```yaml
format: cucumber-json
include: [reports/**/*.json]
exclude: [reports/quarantine/**]
failIfNoResults: true
maxFailedScenarios: 0
outputs: [features, scenarios, failedScenarios]
```

## Productized direction

Keep native JUnit as the supported default. If the plugin is selected, establish ownership, signed Windows/Linux releases, format/version fixtures, and semantic tests. If multiple legacy test formats require conversion, share the cross-platform result-normalization library with NUnit/MSTest while keeping Cucumber-specific gates in a thin adapter.

## Discovery required

- Which Cucumber plugin/version and output format are active?
- Which globs, tags, thresholds, fail-if-no-tests, and report outputs are required?
- Are HTML/report-tab or Jira actions POC blockers?
- Is qTest publication separate from this task?

## Validation

Use representative customer reports and a runnable sample. Verify scenario names/counts, passed/failed/skipped, tags/hooks/backgrounds, include/exclude and recursive globs, empty/malformed files, thresholds, large reports, Windows paths, outputs, task failure, JUnit display, and comparison with Bamboo. Test Jira only if separately selected.

## Effort and ownership

- Native JUnit path: qualification only.
- Bounded `drone-cucumber` repair: 1 to 2 engineering weeks after settings/fixtures exist.
- Jira/report UI/new-format work: separate estimate.
- Likely ownership: CI; Jira integration may involve Platform/External.

## What we can tell the customer

- Harness can display Cucumber scenarios when the runner emits JUnit-compatible results.
- An existing community parser can be repaired if legacy JSON thresholds are required.
- The parser does not execute Cucumber or implement Jira/qTest publication.
- Exact result format and gates determine whether any engineering is needed.

## Sources

- [Cucumber for Bamboo Marketplace listing](https://marketplace.atlassian.com/apps/1214740/cucumber-for-bamboo)
- [Cucumber CI guidance](https://cucumber.io/docs/guides/continuous-integration/)
- [`drone-cucumber` at `a39f074aa8ee6e77e9f17495ace6dc2ab45fd778`](https://github.com/harness-community/drone-cucumber/tree/a39f074aa8ee6e77e9f17495ace6dc2ab45fd778)
- Harness local evidence: `developer-hub` `1c7c98f1d76bb7b8330d6ffba96f984878a32748`, test report reference.
