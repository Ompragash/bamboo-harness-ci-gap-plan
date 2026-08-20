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

Recommendation: provide a governed Harness Git mutation template using a Harness-maintained Windows utility runtime that includes a pinned Git for Windows release.

The template exposes only supported operations and validates the expected base SHA, ref name, signing mode, identity, and push policy. It uses the Harness codebase checkout and a scoped connector or short-lived token. It never logs credentials and does not allow silent force-push.

```text
Harness codebase checkout
-> Harness Windows utility runtime with Git
-> governed commit/tag/branch/push operation
-> remote ref + SHA returned as Harness outputs
```

Proposed inputs: operation, ref, expected SHA, commit/tag message, author identity, signing requirement, remote, and push mode. Standard Git behavior does not require a new Drone plugin. The versioned Harness template provides the structured experience and hides command details.

## What we still need to confirm

- Which commit, merge, branch, tag, signing, and LFS operations are active?
- Which write identity and credential type are approved?
- What protected-branch, force-push, concurrency, and rerun rules apply?

## Customer position

- Harness can provide governed Git writes without embedding scripts in every pipeline.
- Git for Windows will be packaged and maintained by Harness.
- Repository credentials remain scoped Harness secrets/connectors.
- Force-push and signing behavior will follow the customer's repository policy.

## Sources

- [Atlassian source-control task](https://confluence.atlassian.com/bamboo1200/configuring-a-source-control-task-1680480921.html)
- [Harness codebase configuration](https://developer.harness.io/docs/continuous-integration/use-ci/codebase-configuration/create-and-configure-a-codebase/)
- [Harness GitHub App token pattern](https://developer.harness.io/docs/continuous-integration/secure-ci/github-app-token-in-harness/)
