# Repository commit, push, tag, and branch operations

## Customer need

The customer creates branches or tags, commits generated changes, and pushes repository refs from Windows CI. Harness must provide authenticated writes, protected-branch safety, optional signing, predictable reruns, and clear output of the final ref and commit SHA.

## How Bamboo handles it

Bamboo checks out a configured repository on a long-lived agent and retains its repository context and credentials. Source-control tasks use that context to commit modified files, push existing or custom commits, create branches/tags from the checkout commit, and push the ref. Bamboo can also support signed commits and Git LFS in selected modes.

```text
Bamboo checkout with repository credentials
-> commit / branch / tag / push task
-> installed Git mutates remote repository
-> result returns to Bamboo
```

## Harness implementation

Recommendation: enable **Persist Credentials** on Harness **Clone Codebase**, then use the existing Windows-capable `harness/drone-git` image in a native Run step for the required commit, branch, tag, or push operation. No new Windows Git image or Git plugin is required.

Harness documents Persist Credentials specifically for Git commands such as tagging and pushing after clone. On Windows, the clone implementation copies its `_netrc` credentials to the build's shared credential location, and Harness restores them for later steps in the same build. The credentials are not retained after the build finishes.

```text
Harness Clone Codebase with Persist Credentials enabled
-> authenticated checkout and credentials retained for this build only
-> Run step with a qualified harness/drone-git Windows image
-> governed commit/tag/branch/push operation
-> remote ref + SHA returned as Harness outputs
```

`harness/drone-git` is Harness's clone image, and its Windows variants already contain Git for Windows, Git LFS, OpenSSH, and PowerShell. It is not a separate commit/tag/push plugin; the later Run step uses the Git executable already present in that image. The POC must use a qualified image tag or digest that matches the Windows worker's LTSC version.

A reusable Harness Run Step Template can provide the structured customer experience without duplicating commands in every pipeline. The template can expose operation, ref, expected SHA, commit/tag message, author identity, signing requirement, remote, and push mode while enforcing the customer's branch and force-push policies.

Persist Credentials reuses the credential already configured on the selected codebase connector; it does not grant additional repository permissions or automatically select another identity. The selected connector's credential must therefore have the required write scope for the POC.

## What we still need to confirm

- Which commit, merge, branch, tag, signing, and LFS operations are active?
- What protected-branch, force-push, concurrency, and rerun rules apply?
- Which Windows LTSC worker version will the POC use so Harness can qualify the matching `harness/drone-git` image tag or digest?

## Customer position

- Native Clone Codebase performs checkout and can persist its credentials for authenticated Git operations later in the same build.
- Harness will reuse the existing Windows `harness/drone-git` image; the customer does not need a new custom Git image or a new Git plugin.
- A governed Run Step Template can standardize Git writes without duplicating commands across pipelines.
- Persisted credentials remain scoped to the build and retain only the permissions of the configured repository identity.
- Force-push and signing behavior will follow the customer's repository policy.

## Sources

- [Atlassian source-control task](https://confluence.atlassian.com/bamboo1200/configuring-a-source-control-task-1680480921.html)
- [Harness codebase configuration](https://developer.harness.io/docs/continuous-integration/use-ci/codebase-configuration/create-and-configure-a-codebase/)
- [Use codebase connector Git credentials in a CI Run step](https://developer.harness.io/docs/continuous-integration/ci-articles-faqs/articles/use-git-credentials-from-codebase-connector-in-ci-pipelines-run-step/)
- [Harness CI images](https://developer.harness.io/docs/continuous-integration/use-ci/set-up-build-infrastructure/harness-ci/)
- [Harness Cloud Windows build infrastructure](https://developer.harness.io/docs/continuous-integration/use-ci/set-up-build-infrastructure/use-harness-cloud-build-infrastructure/)
- [Harness GitHub App token pattern](https://developer.harness.io/docs/continuous-integration/secure-ci/github-app-token-in-harness/)
