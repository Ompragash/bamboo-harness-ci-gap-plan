# qTest result publication

## Customer need

The customer publishes JUnit test results from Bamboo into Tricentis qTest. Harness must preserve qTest authentication, project/release/environment selection, result-file matching, test-case grouping, submission status, and publication failure behavior.

Harness test reporting and qTest publication are separate. Tests should remain visible in Harness while the same results are sent to qTest.

## How Bamboo handles it

Tricentis documents that the Bamboo plugin does not execute tests. It runs after the test task, scans JUnit XML using an optional Ant-style pattern, connects to qTest with an integration token, selects the project/release and optional environment, submits test runs/logs, and reports submission status.

```text
Bamboo test task produces JUnit XML
-> qTest task maps project/release/environment
-> results submitted to qTest
-> submission status returned to Bamboo
```

## Harness implementation

Recommendation: build a Harness qTest Publisher plugin as one fixed Harness-maintained Windows integration image. API authentication, object mapping, submission, polling, and outputs justify a Plugin step.

The plugin runs after Harness has ingested the JUnit report. It validates the endpoint and result glob, resolves the selected qTest objects, submits the JUnit results, waits for a terminal submission state when the API is asynchronous, and returns the qTest submission ID, state, and URL as Harness outputs.

```text
Harness test step produces and ingests JUnit
-> Plugin step references harness/qtest-publisher:windows-<ltsc>
-> qTest project/release/environment lookup
-> submit results and wait for status
-> Harness outputs and configured failure result
```

Initial inputs: endpoint, authentication secret, project, release, optional environment, JUnit glob, suite-versus-test-case grouping, and fail-on-publication-error. Proxy/private CA, bounded retry for documented transient errors, idempotency, timeout, cancellation, and secret-safe logs are required.

The pipeline references the Plugin image tag explicitly. qTest settings are passed to that image and do not affect image selection. Harness builds, signs, scans, publishes, tests, and maintains it. Implementation starts only after a non-production qTest tenant proves the customer API and mapping contract.

## What we still need to confirm

- Which qTest product/version and authentication method are used?
- How are project, release, environment, suite/cycle, and JUnit grouping mapped?
- Should missing objects be created, and should publication failure fail CI?
- Is a non-production qTest tenant available for end-to-end tests?

## Customer position

- Harness can continue native test reporting and publish the same JUnit results to qTest.
- qTest publication requires a dedicated integration plugin because it performs authenticated object mapping and submission.
- Harness will own the plugin image and support policy.
- A test-tenant proof is required before the implementation commitment is finalized.

## Sources

- [Tricentis Jenkins and Bamboo integration](https://docs.tricentis.com/qtest-2026.2/content/integrations/jenkins_and_bamboo_integration.htm)
- [Tricentis qTest Bamboo compatibility](https://docs.tricentis.com/qtest-2026.2/content/integrations/tricentis_product_and_integrating_app_compatibility_od.htm)
- [Tricentis Test Log APIs](https://docs.tricentis.com/qtest-latest/content/apis/apis/test_log_apis.htm)
