# Xcode keychain and signing access

## Customer need

The customer builds with Xcode 14.3 and unlocks a macOS keychain so Xcode or `codesign` can access a signing certificate and private key.

This cannot run in a Windows container. Apple keychains, Xcode, provisioning profiles, and Apple signing tools require macOS execution.

## How Bamboo handles it

Bamboo selects a long-lived macOS agent with Xcode and the keychain already present. The Xcode Unlock Keychain task wraps the macOS `security` command, supplies the protected password, and makes the identity available to later Xcode or codesign tasks.

```text
Bamboo selects macOS agent with Xcode and keychain
-> unlock task uses protected password
-> Xcode/codesign accesses identity
-> build continues on macOS
```

## Harness implementation

Recommendation: use a native Run step on a Harness-managed macOS execution environment with an ephemeral-keychain template. No Windows or keychain plugin is required.

The template creates a temporary keychain, imports the certificate/private key and provisioning profile from Harness secrets, configures keychain search/access settings, executes Xcode signing, and always locks and deletes the temporary keychain.

```text
Harness Run step on macOS runner with Xcode 14.3
-> create temporary keychain
-> import non-production signing identity/profile
-> xcodebuild or codesign
-> always lock and delete keychain
```

No Windows runtime image or plugin can replace this operating-system dependency. The Windows Kubernetes POC therefore needs a separate macOS lane if Xcode is a required acceptance case.

## What we still need to confirm

- Which Harness-managed macOS runner image will provide Xcode 14.3?
- Does Bamboo unlock an existing keychain or import an ephemeral certificate/profile?
- Which signing identity, profile, concurrency, timeout, and cleanup rules are required?

## Customer position

- Harness can provide the signing workflow, but it must run on macOS.
- The Windows Kubernetes environment cannot execute Xcode or Apple keychain operations.
- Harness will provide a reusable secret-safe keychain lifecycle with guaranteed cleanup.

## Sources

- [Apple Keychain Access guide](https://support.apple.com/guide/keychain-access/kychn001/mac)
- [Apple keychain data protection](https://support.apple.com/guide/security/keychain-data-protection-secb0694df1a/web)
- [Harness iOS CI guide](https://developer.harness.io/docs/continuous-integration/development-guides/mobile/ios/)
