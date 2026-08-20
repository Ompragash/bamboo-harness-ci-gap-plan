# Repository tag, commit, push, and branch

| Field | Value |
| --- | --- |
| Bamboo plugin key | plugins.vcs:task.vcs.tagging, plugins.vcs:task.vcs.commit, plugins.vcs:task.vcs.push, plugins.vcs:task.vcs.branching |
| Provider | Bamboo native |
| Customer version(s) | Not provided |
| Harness CSV status | No |
| Scope | CI governed reusable template |
| Recommended Harness approach | Native checkout plus a governed Git-for-Windows Run step template |
| Solution type | D. Reusable Harness template |
| Discovery required | Yes |
| Planning confidence | Medium |

## 1. What this Bamboo task does

These tasks mutate a source repository after checkout: create a branch or tag, commit generated changes, and push refs. Bamboo wraps standard VCS operations with repository context and credentials.

The meaningful requirements are authenticated writes, safe ref selection, auditability, and predictable failure behavior.

## 2. How it works in Bamboo

Bamboo checkout → VCS mutation task → configured repository and credentials → tag/branch/commit/push → remote response and task status.

Material inputs include operation, ref name, commit message, author identity, remote, credentials, and sometimes working copy changes. Server branch protection and permissions still apply.

## 3. How the customer uses it

Confirmed customer usage: the inventory groups tagging, commit, push, and branching tasks. No remote, signing, LFS, generated-file, or branch-policy detail is included.

Typical plugin capability: create release tags, commit version changes, create release branches, or push build-generated updates.

Customer usage context: not confirmed from the available source material.

Smallest question: Which exact operations and remotes are used, and do they require signed tags, force-push, LFS, submodules, or protected-branch bypass?

## 4. What Harness supports today

Harness has native Clone Codebase/Git clone behavior and connector-managed credentials. The codebase configuration includes Persist Credentials for later Git operations when the clone identity is intentionally write-enabled. Git for Windows can perform standard mutations in a Run step. Harness also documents a short-lived GitHub App token plugin and a Git revert plugin, confirming a separate least-privilege write-token path and specialized operations.

RunStepInfo verifies that the step contract manages environment, secrets, outputs, reports, resources, image, shell, and failure lifecycle. The command is governed pipeline configuration, not an unmanaged execution channel.

The CSV says No because there is no one native task type covering all four mutations. The underlying secure workflow does not require a new binary plugin.

## 5. The actual gap

The gap is a reusable, policy-aware template that standardizes inputs, identity, credentials, ref validation, output variables, and audit logs on Windows. Specialized signed-tag or arbitrary-remote behavior may change the design.

## 6. Recommended Harness solution

Recommendation: use native checkout and a versioned Git-for-Windows Run template for approved tag, branch, commit, and push operations.

The template exposes an allow-listed operation, ref, message, author, remote, and changed paths. It either uses deliberately persisted clone-connector credentials or accepts a separate short-lived write token from a Harness secret, emits the created commit/ref, masks credentials, and fails on remote rejection.

Engineering work is template logic, guardrails, examples, and qualification. We should not build a generic Git plugin that hides standard Git behavior and server-side policy.

Result: teams get a reusable platform pattern rather than copy-pasted commands, with native Harness RBAC, logs, secrets, and audit.

## 7. Proposed implementation shape

- Step: PowerShell Run step on an image containing a pinned Git for Windows.
- Inputs: operation, branch/tag, source ref, remote allow-list, commit message, author, paths, push mode, signing mode.
- Credentials: persisted clone credentials only when that identity is intentionally write-enabled; otherwise a short-lived GitHub App/token or approved SSH key. Never put credentials in remote URL output.
- Outputs: resulting commit SHA and ref.
- Guardrails: no force-push by default, reject unsafe ref names, explicit opt-in for protected operations, clean working-tree checks.
- Validation: GitHub/GitLab/Bitbucket variant actually used by the customer.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which operations and remotes are active? | Defines the template surface. | Customer |
| Are signed commits/tags, LFS, submodules, or force-push required? | May need extra tooling or policy. | Customer |
| What write credential model is approved? | Determines connector/token handling and rotation. | Customer / Security |
| Which branch protections must remain enforced? | Defines negative tests and guardrails. | Customer |

## 9. Validation plan

Use a disposable repository under the customer's SCM policy. Create and push a branch and tag, commit an allowed generated file, verify outputs, and prove remote rejection and protected-branch behavior. Test paths with spaces, credentials masking, token expiry, retry after network failure, and cleanup.

## 10. Dependencies and risks

- Blocking: no approved write credential or disposable repository.
- Planning: signed refs and arbitrary remotes materially expand scope.
- Implementation: credentials leakage and Windows quoting.
- Long-term maintenance: SCM provider token and branch-policy changes.

## 11. Planning estimate

<1 engineering week for the standard checked-out-repository case with unsigned branch/tag/commit/push. Discovery is required before estimating signing, LFS, or policy-bypass cases.

## 12. What we can tell the customer now

- Harness can perform governed repository mutations using native checkout and scoped write credentials.
- A reusable template can standardize the experience and avoid pipeline-by-pipeline command maintenance.
- Standard Git operations do not justify a new plugin.
- We need the write operations, credential model, and branch policies before finalizing the template.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 14.

### Bamboo/vendor

- [Atlassian: Configuring a source control task](https://confluence.atlassian.com/bamboo1200/configuring-a-source-control-task-1680480921.html)
- [actions/checkout documentation](https://github.com/actions/checkout/blob/main/README.md)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/secure-ci/github-app-token-in-harness.md
- [Harness: Create and configure a codebase](https://developer.harness.io/docs/continuous-integration/use-ci/codebase-configuration/create-and-configure-a-codebase/)
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/codebase-configuration/git-revert-commit.md
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/platform/templates/run-step-template-quickstart.md
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/RunStepInfo.java

Confidence: Medium.
