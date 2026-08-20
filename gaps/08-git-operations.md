# Repository commit, push, tag, and branch operations

## Customer need

The customer creates branches or tags, commits generated changes, and pushes repository refs from CI. The active operations, write identity, signing, branch protection, Git LFS, and concurrency policy are unknown.

The important outcome is a governed authenticated mutation with clear provenance and safe rerun behavior.

## What Bamboo provides

Bamboo source-control tasks operate on a selected/default checked-out repository and reuse Bamboo repository credentials. The commit task commits modified files and pushes. Push can publish existing, custom, signed, or merge commits and can push multiple commits transactionally. Branch/tag tasks create a ref from the latest checkout commit and push it. Official docs also describe Git LFS support.

```text
Bamboo checkout and repository context
-> branch/tag/commit/push task
-> stored repository credentials
-> remote ref mutation and task result
```

## Harness today

Harness has native codebase checkout, optional persisted credentials, secret/connectors, GitHub App token patterns, Run steps, failure strategies, and reusable templates. Git for Windows performs the actual mutation.

## Gap

Harness lacks a single governed template that exposes only approved mutations and preserves Bamboo's convenient repository/credential context. An arbitrary inline Git script would work technically but could duplicate credential handling, validation, signing, and retry policy.

## Recommended approach

Recommendation: provide a versioned Git-for-Windows Run template, not a new Git plugin, for the selected operations.

The template validates operation and ref names, checks the expected base SHA, configures a short-lived identity, uses either persisted checkout credentials or a separate scoped token, avoids logging secrets, and fails on non-fast-forward or signing errors. It must never silently force-push. A product step is only justified later if structured repository write policy and outputs cannot be expressed safely in a template.

## POC experience

Proposed template inputs, not final Harness YAML:

```yaml
operation: tag
ref: release/1.2.3
expectedSha: <+codebase.commitSha>
message: Release 1.2.3
sign: false
pushMode: fast-forward-only
```

Use a disposable repository to demonstrate one active operation, duplicate/rerun handling, protected branch behavior, and a deliberately rejected stale SHA.

## Productized direction

Keep the template if the required behavior remains standard Git. Add typed outputs for pushed ref and SHA. Consider a first-class repository mutation step only if multiple customers require connector-level policy, server-side idempotency, or native signing/approval workflows beyond a template.

## Discovery required

- Which commit, push, branch, tag, merge, LFS, and signed-ref modes are active?
- Which service identity and credential lifetime are approved?
- What branch protection, concurrency, rerun, and force-push rules apply?

## Validation

Test success, duplicate tag/branch, no changes, stale base SHA, non-fast-forward, protected branch, token expiry, signing if required, LFS if active, cancellation, output SHA/ref, and secret masking. Confirm remote history matches Bamboo's intended result.

## Effort and ownership

- POC and supported template: less than 1 engineering week for standard operations.
- Likely ownership: CI + SCM/platform security.

## What we can tell the customer

- Harness can perform authenticated repository writes through native checkout, secrets/connectors, and governed steps.
- The POC will use a reusable policy-controlled template rather than copy Git commands into every pipeline.
- Exact signing and branch policies must be confirmed before enabling writes.

## Sources

- [Atlassian source-control task](https://confluence.atlassian.com/bamboo1200/configuring-a-source-control-task-1680480921.html)
- [Bamboo Specs task reference](https://docs.atlassian.com/bamboo-specs-docs/10.0.2/specs.html?yaml=)
- [Harness codebase configuration](https://developer.harness.io/docs/continuous-integration/use-ci/codebase-configuration/create-and-configure-a-codebase/)
- [Harness GitHub App token pattern](https://developer.harness.io/docs/continuous-integration/secure-ci/github-app-token-in-harness/)
