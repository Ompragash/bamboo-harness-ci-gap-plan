# Xcode keychain and signing access

## Customer need

The customer builds with Xcode 14.3 and needs signing tools to access a certificate/private key protected by a macOS keychain. The runner type, keychain source, certificate/profile import, timeout, signing identity, cleanup, and concurrency model are unknown.

This capability is part of macOS CI. There is no Windows equivalent because Apple keychains and codesigning identities are provided by macOS Security.framework and Apple tooling.

## What Bamboo provides

Bamboo's Xcode plugin supplies an Unlock Keychain task that wraps the macOS `security` command and secret handling so later Xcode or `codesign` invocations can access the signing identity.

```text
macOS agent
-> Bamboo unlock task with protected password
-> user or temporary keychain unlocked
-> Xcode/codesign reads identity
```

The convenience is credential-safe setup in the correct runner context. The underlying keychain remains host-level macOS state.

## Harness today

Harness can execute macOS CI commands, inject secrets, use templates, and run the documented certificate/profile setup in an iOS pipeline. A Run step on a macOS runner can create or unlock a keychain, import a PKCS#12 identity, set search order and access controls, build/sign, and clean up.

## Gap

The gap is a qualified, reusable keychain lifecycle for the customer's Xcode 14.3 runner and signing model. Running keychain operations on a Windows Kubernetes node is impossible and would not make Xcode available. If the customer's primary CI infrastructure is Windows Kubernetes, a separate macOS execution pool must be part of the POC topology.

## Recommended approach

Recommendation: use a macOS runner and an ephemeral-keychain template with explicit create/import/unlock/use/cleanup phases.

Prefer a new per-execution keychain over unlocking a long-lived login keychain. Import the minimum identity, set a bounded timeout, make the keychain available only to required tools, avoid parallel reuse, and delete it in an always-run cleanup step. Keep certificate bytes, passwords, and provisioning data in Harness secrets or an approved secret store.

## POC experience

```text
macOS runner with Xcode 14.3
-> create temporary keychain
-> import non-production signing certificate/profile
-> configure search list and partition/access settings
-> xcodebuild or codesign
-> always lock and delete temporary keychain
```

Proposed template inputs, not final Harness YAML: Xcode runner selector, keychain name, PKCS#12 secret, password secret, provisioning profile secret/path, signing identity, timeout, and cleanup mode.

## Productized direction

Publish a supported macOS signing template and runbook with secret-store patterns, runner isolation requirements, concurrency guidance, redacted diagnostics, and cleanup guarantees. A dedicated plugin is unnecessary unless native signing asset lifecycle or certificate-store integration becomes a broader product need.

## Discovery required

- Which macOS runner and Xcode 14.3 image are available?
- Is the current flow unlocking an existing keychain or importing an ephemeral certificate/profile?
- Which signing identity, profile, keychain timeout, concurrency, and cleanup behavior are required?

## Validation

Use non-production signing material. Verify identity discovery, signed artifact validation, wrong-password failure, missing profile, concurrent jobs, cancellation and always-run cleanup, redacted logs, keychain deletion, and no residual private key on the runner.

## Effort and ownership

- POC: qualification only if runner and assets exist; less than 1 engineering week for a reusable template.
- Likely ownership: CI + Platform/macOS infrastructure; signing assets remain customer-owned.

## What we can tell the customer

- Harness can perform Xcode keychain and signing setup on a macOS runner with governed secrets and cleanup.
- The POC needs a macOS execution pool even when the rest of CI runs on Windows Kubernetes.
- An ephemeral keychain reduces shared-state and credential-residue risk.

## Sources

- [Apple Keychain Access guide](https://support.apple.com/guide/keychain-access/kychn001/mac)
- [Apple Platform Security keychain protection](https://support.apple.com/guide/security/keychain-data-protection-secb0694df1a/web)
- [Harness iOS CI guide](https://developer.harness.io/docs/continuous-integration/development-guides/mobile/ios/)
