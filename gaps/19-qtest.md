# qTest result publication

## Customer need

The customer publishes automated test results from Bamboo to Tricentis qTest. The qTest product/version, endpoint, authentication, project/release/environment, result glob, suite-versus-case grouping, overwrite behavior, failure policy, and test-tenant availability are unknown.

Tests should remain visible in Harness, while qTest receives the external test-management record required by the customer.

## What Bamboo provides

Tricentis documents that the Bamboo plugin does not execute tests. It scans JUnit XML, connects with a qTest URL and user-level integration token, selects a project and release, optionally assigns an environment, submits results, and provides submission status/failure reporting. For Bamboo, only JUnit XML is supported and the result path can use an Ant-style pattern.

```text
test runner and JUnit XML
-> qTest Bamboo task
-> project/release/environment mapping
-> qTest test runs/logs and submission status
```

Current Tricentis docs show a Bamboo Data Center 10-compatible release, but customer version/API behavior still requires confirmation. Public Bamboo plugin implementation source was not located.

## Harness today

Harness can run tests and ingest JUnit natively. Its qTest AIDI/SEI integration reads qTest data for analytics; it is not a CI-side result publisher. No Harness/Drone qTest publisher was found in the reviewed community/plugin inventory.

## Gap

The missing behavior is structured external synchronization: qTest authentication and object lookup, JUnit mapping/grouping, submission, asynchronous status if applicable, retry/idempotency, and clear CI failure policy. Repeated API scripts in Run steps would expose integration complexity and credentials in every pipeline.

## Recommended approach

Recommendation: perform a read-only plus test-submission proof in a non-production qTest tenant, then build a Windows-capable publisher plugin only if the required mapping is confirmed.

The first bounded contract should cover endpoint/auth, project, release, optional environment, Ant-style JUnit glob, suite-versus-test-case grouping if required, and fail-on-publication policy. Add build/cycle/suite creation, overwrite steps, attachments, and advanced grouping only when exported settings require them. The plugin publishes results after native Harness ingestion; it does not execute tests.

## POC experience

Proposed user-facing inputs, not final Harness YAML:

```yaml
endpoint: https://company.qtestnet.com
authSecret: qtest-integration-token
project: Product
release: POC
environment: Windows-LTSC2022
resultGlob: TestResults/**/*.xml
grouping: junit-suite
failOnPublishError: true
```

Before implementation, the POC proof retrieves allowed projects/releases/environments and submits one representative JUnit report to a disposable target.

## Productized direction

If selected, create a small cross-platform publisher with Windows LTSC packaging, signed releases, typed lookup/submission outputs, terminal-status polling when the API is asynchronous, bounded retry/backoff only for documented transient errors, idempotency/correlation keys, proxy/private CA, secret-safe logs, and mock plus tenant E2E tests. Maintain a documented qTest compatibility policy and vendor escalation path.

Do not couple qTest publication to the Cucumber/NUnit/MSTest normalizer. qTest consumes the final JUnit contract after Harness test reporting.

## Discovery required

- Which qTest product/version, endpoint, authentication, and API are used?
- How are project, release, environment, suite/cycle, and JUnit grouping mapped?
- What result glob, overwrite, attachment, object-creation, and publication-failure behavior is active?
- Is a non-production tenant and least-privilege service account available?

## Validation

Use a non-production project. Verify lookup, passed/failed/skipped mapping, suite-versus-case grouping, repeated submission/idempotency, terminal status/polling, documented transient retry, timeout, bad credentials, malformed/empty JUnit, missing objects, private CA/proxy, Windows execution, outputs, secret masking, and configured CI failure behavior. Compare created qTest records with Bamboo.

## Effort and ownership

- POC API/test-tenant proof: discovery required, likely less than 1 engineering week after access exists.
- Bounded publisher plugin: 2 to 4 engineering weeks after the proof and ownership decision.
- Likely ownership: CI + External/vendor; qTest API support depends on Tricentis.

## What we can tell the customer

- Harness will continue to run tests and display JUnit results natively.
- qTest publication is a separate structured integration and is the clearest new-plugin candidate in this plan.
- A test-tenant proof and exported mappings are required before an implementation commitment.
- The first scope will stay bounded to the qTest behavior the customer actually uses.

## Sources

- [Tricentis Jenkins and Bamboo integration](https://docs.tricentis.com/qtest-2026.2/content/integrations/jenkins_and_bamboo_integration.htm)
- [Tricentis qTest Bamboo compatibility](https://docs.tricentis.com/qtest-2026.2/content/integrations/tricentis_product_and_integrating_app_compatibility_od.htm)
- [Tricentis qTest Bamboo Marketplace history](https://marketplace.atlassian.com/apps/1214423/tricentis-qtest-integration-for-bamboo/version-history)
- [Tricentis Test Log APIs](https://docs.tricentis.com/qtest-latest/content/apis/apis/test_log_apis.htm)
- Harness local evidence: `developer-hub` `1c7c98f1d76bb7b8330d6ffba96f984878a32748`, qTest analytics and test-report docs.
