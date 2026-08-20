# qTest

| Field | Value |
| --- | --- |
| Bamboo plugin key | qtest-plugin-for-bamboo |
| Provider | Tricentis |
| Customer version(s) | Not provided |
| Harness CSV status | No |
| Scope | CI and test management |
| Recommended Harness approach | Possible Windows-capable result-publishing plugin after API/test-tenant proof and ownership decision |
| Solution type | G. New plugin candidate, gated by discovery |
| Discovery required | Yes |
| Planning confidence | Low |

## 1. What this Bamboo task does

The qTest Bamboo integration finds JUnit XML produced by tests and submits results into a qTest project hierarchy. It maps results to release/environment and related objects, creates test runs/logs, and uploads failure information.

It does not execute tests. Its distinctive value is remote object resolution or creation, result translation, batching, authentication, retry, and synchronization with qTest.

## 2. How it works in Bamboo

Bamboo test job emits JUnit XML → qTest plugin matches files → resolves qTest project/release/environment and other mappings → creates or updates test runs/logs through qTest APIs → returns publication status to Bamboo.

Material inputs can include qTest URL, credentials, project, release, build/cycle/environment, grouping, Ant-style result glob, and attachment behavior.

## 3. How the customer uses it

Confirmed customer usage: qTest is in the inventory. No qTest edition/version, URL, auth model, hierarchy, mappings, result globs, attachments, or failure policy are supplied.

Typical plugin capability: publish JUnit automation results to qTest after CI execution.

Customer usage context: not confirmed from the available source material.

Smallest question: Provide one exported qTest task and confirm the qTest version/URL, authentication, project hierarchy mappings, result glob, and attachment/failure behavior.

## 4. What Harness supports today

Harness natively ingests JUnit into the Tests tab. A documented qTest integration exists in Harness AIDI/SEI for reading qTest Cloud data into analytics, but that is not CI-side result publication.

No existing Harness/Drone qTest result publisher was found in the local and official/community repository review performed on 2026-08-20. qTest supplies APIs and Bamboo/Jenkins integration semantics.

The CSV says No because Harness does not currently perform the qTest object mapping and result synchronization.

## 5. The actual gap

Test execution and Harness visibility already exist. The minimum missing behavior is submitting JUnit outcomes to a confirmed qTest project/release and optional environment, polling the asynchronous submission to a terminal state, and returning a clear publication status.

This is one row where a plugin is potentially justified, but the API contract and customer configuration must be known before scope or effort is confirmed.

## 6. Recommended Harness solution

Recommendation: treat a Windows-capable result-publishing plugin as the current candidate only after P0 discovery, a read-only/test-tenant API proof, and an ownership decision confirm qTest synchronization is required for the POC.

For the first bounded contract, the customer configures qTest endpoint and secret, project, release, optional environment, JUnit glob, and publication-failure policy. Build/cycle/grouping, attachments, and object creation are added only if the exported task proves they are required. Harness supplies Plugin-step governance, secrets, logs, timeout, failure strategy, outputs, RBAC, and audit.

Engineering validates one auth model and implements project/release/environment lookup, JUnit submission, asynchronous job-status polling, bounded retry/backoff, safe diagnostics, structured status outputs, and Windows packaging. We should not put qTest publication into a language image or require every pipeline to maintain API scripts.

Result: CI remains responsible for tests, while a reusable integration publishes the same results into qTest.

## 7. Proposed implementation shape

- Vendor interface: qTest REST/bulk automation-log APIs validated against the customer's version.
- Inputs for the bounded candidate: endpoint, auth secret, project, release, optional environment, result glob, and fail-on-publish policy.
- Outputs: asynchronous submission job ID, terminal state, submitted result count when returned, and publication URL/identifier.
- Reliability: submit once, poll to terminal state with timeout, bounded retry/backoff for documented transient errors, and cancellation.
- Security: TLS/private CA/proxy, secret masking, least-privilege service account, no result data in debug logs.
- Platform: small Go or supported-JRE Windows LTSC plugin image; no application language SDKs required. Add Linux packaging only after the first contract is proven.
- Tests: mock API contract plus qTest test-tenant E2E for lookup, submit-and-poll, retry, timeout, malformed JUnit, and API errors.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which qTest product/version, endpoint, and authentication model are used? | Defines the API and credential contract. | Customer / Vendor |
| How are project, release, build/cycle, environment, and grouping mapped? | Defines object resolution and creation. | Customer |
| What JUnit glob, attachments, and failure information are submitted? | Defines parsing and payload scope. | Customer |
| Should missing objects be created, and should publication failure fail CI? | Defines permissions and result semantics. | Customer / Product |
| Is a non-production qTest tenant available? | Required before an implementation commitment. | Customer |

## 9. Validation plan

Use a non-production qTest project and representative JUnit files. Verify project/release/optional-environment lookup, passed/failed/skipped results, asynchronous submit-and-poll, timeout, documented transient retry, bad credentials, malformed JUnit, API failure, private CA/proxy, Windows Kubernetes, secret masking, and configured CI failure behavior. Compare created runs/logs with Bamboo. Test attachments or object creation only if exported fields require them.

## 10. Dependencies and risks

- Blocking: exported task, API documentation for the customer's version, service account, and test tenant.
- Planning: hierarchy creation and attachments can materially expand scope.
- Implementation: rate limits, idempotency, partial failure, and result mapping.
- Long-term maintenance: qTest API/version changes and plugin ownership.

## 11. Planning estimate

Discovery required before estimate. Do not apply a 1 to 2 engineering week implementation estimate until the exported Bamboo fields, qTest API/version, asynchronous submission contract, test tenant, and ownership are verified.

## 12. What we can tell the customer now

- Harness can run the tests and display JUnit results natively.
- qTest publication is a separate structured integration need and is the clearest new-plugin candidate in this CI set.
- We need the qTest hierarchy mappings, authentication, and a test tenant before committing to an implementation scope.
- The plugin would publish results; it would not replace the native Harness Test step.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 33.

### Bamboo/vendor

- [Tricentis: Jenkins and Bamboo integration](https://docs.tricentis.com/qtest-2026.2/content/integrations/jenkins_and_bamboo_integration.htm)
- [Tricentis: Scaling test automation with qTest APIs](https://docs.tricentis.com/qtest-2026.2/content/apis/customer_case_studies/dolby_scaling_test_automation_with_qtest_apis.htm)
- [Tricentis: Test Log APIs](https://docs.tricentis.com/qtest-latest/content/apis/apis/test_log_apis.htm)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/run-tests/test-report-ref.md
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/ai-dlc-insights/setup/integrations/qtest/index.md
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/PluginStepInfo.java

Confidence: Low.
