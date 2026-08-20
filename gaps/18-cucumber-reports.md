# Cucumber results

## Customer need

The customer processes Cucumber results after test execution. Harness must show scenario results and preserve any required include/exclude patterns, tags, no-result policy, failure thresholds, and optional Jira behavior.

This task processes reports. The language-specific Cucumber runner remains responsible for executing tests.

## How Bamboo handles it

The Bamboo Cucumber plugin reads result files produced by the test runner, turns features/scenarios into build-visible results, applies configured gates, and may add a report tab or Jira actions depending on the plugin version.

```text
Cucumber runner produces report
-> Bamboo Cucumber task parses scenarios and thresholds
-> Bamboo test/report view and task result
```

## Harness implementation

Recommendation: use native Harness JUnit reporting when the Cucumber runner can emit JUnit; harden the existing `drone-cucumber` plugin only when the customer's JSON thresholds or tag behavior require it.

The preferred flow is:

```text
Cucumber runner in Harness language runtime
-> JUnit report
-> Harness native test reporting
```

If legacy Cucumber JSON processing is required, Harness publishes a repaired Harness Cucumber Results plugin image based on the community repository. Required changes are:

- apply exclude patterns correctly;
- implement recursive `**` globs;
- count failed scenarios/features rather than failed steps for those thresholds;
- fail clearly on malformed or missing results according to policy;
- write stable counts as Harness outputs;
- add Windows LTSC tests, signed image publication, and release ownership.

The plugin remains a parser and gate. It does not execute Cucumber, generate Jira issues by default, or publish to qTest. Jira actions are added only after a separate integration decision.

## What we still need to confirm

- Which Cucumber output format and plugin version are active?
- Which globs, tags, thresholds, and no-result settings are required?
- Can the runner emit JUnit, or is legacy JSON mandatory?
- Are Jira actions or a dedicated report tab POC requirements?

## Customer position

- Harness can display Cucumber scenarios through native JUnit reporting.
- Harness will repair and maintain the existing community parser only if legacy JSON gates are required.
- The Cucumber parser will not duplicate test execution or qTest publication.

## Sources

- [Cucumber for Bamboo Marketplace listing](https://marketplace.atlassian.com/apps/1214740/cucumber-for-bamboo)
- [Cucumber CI guidance](https://cucumber.io/docs/guides/continuous-integration/)
- [`drone-cucumber`](https://github.com/harness-community/drone-cucumber)
- [Harness test report reference](https://developer.harness.io/docs/continuous-integration/use-ci/run-tests/test-report-ref/)
