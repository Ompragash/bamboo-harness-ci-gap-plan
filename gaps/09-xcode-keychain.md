# Xcode unlock keychain

| Field | Value |
| --- | --- |
| Bamboo plugin key | bamboo-xcode-plugin:unlockkeychain |
| Provider | Bamboo native Xcode plugin |
| Customer version(s) | Xcode 14.3 |
| Harness CSV status | No |
| Scope | macOS CI, outside Windows execution |
| Recommended Harness approach | Reusable macOS keychain/signing runbook and Run step template |
| Solution type | B. Existing native capability plus qualification |
| Discovery required | Yes |
| Planning confidence | Medium |

## 1. What this Bamboo task does

The task unlocks a macOS keychain so Xcode or codesign can access signing identities during an Apple build. Bamboo supplies a convenient task wrapper and secret handling around Apple's security command and keychain service.

It is inherently macOS-specific. A keychain used for Xcode signing is part of Apple Security.framework and has no Windows counterpart.

## 2. How it works in Bamboo

macOS Bamboo agent → unlock-keychain task → macOS security service and stored keychain → Xcode/codesign uses the identity → later cleanup or lock.

Material inputs are keychain path and password. A complete ephemeral signing flow can also import certificates, install provisioning profiles, adjust the keychain search list and partition list, then remove temporary material.

## 3. How the customer uses it

Confirmed customer usage: the inventory specifies Xcode 14.3 and the unlock-keychain task. This is a macOS CI requirement, not a Windows Kubernetes workload.

Typical plugin capability: unlock an existing keychain before build/signing.

Customer usage context: not confirmed from the available source material.

Smallest question: Does the current job only unlock a pre-provisioned keychain, or also import certificates/profiles and delete temporary signing material?

## 4. What Harness supports today

Harness documents macOS build infrastructure and direct Apple security commands for creating, unlocking, importing into, and listing a keychain. Harness secrets can supply certificate material and passwords, while a Run step manages logs, environment, timeout, failure strategy, and cleanup commands.

The CSV says No because there is no dedicated unlock-keychain task type. The underlying operation is supported on macOS, and Windows cannot provide the Apple service regardless of plugin packaging.

## 5. The actual gap

The gap is a qualified, reusable signing setup/cleanup pattern with safe secret handling on the chosen macOS runner. It is not a missing Windows image or Windows plugin.

Xcode 14.3 also constrains the macOS/Xcode runner image. That runner availability and certificate workflow need confirmation.

## 6. Recommended Harness solution

Recommendation: provide a versioned macOS Run step template and runbook for an ephemeral keychain and signing assets.

The customer selects or creates a temporary keychain, passes encrypted certificate/profile material through Harness secrets, configures the search list, runs Xcode or Fastlane, and deletes the temporary keychain in cleanup.

Engineering work is qualification, safe log handling, template inputs, cleanup behavior, and an Xcode 14.3 runner check. We should not build a Windows plugin because it cannot implement Apple's keychain or codesign services.

Result: a repeatable macOS signing workflow with Harness governance and minimal persistent secret material.

## 7. Proposed implementation shape

- Infrastructure: customer-managed or supported macOS runner with Xcode 14.3.
- Steps: create temporary keychain if needed, unlock, import identity, install provisioning profile, set search/partition access, build/sign, always clean up.
- Secrets: keychain password, certificate password, certificate/profile content.
- Outputs: non-sensitive signing identity metadata and artifact path only.
- Failure strategy: cleanup must run after build failure or cancellation where the runner supports it.
- Runbook: pre-provisioned versus ephemeral modes, rotation, runner isolation, and log redaction.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Is the keychain pre-provisioned or created per build? | Defines setup and cleanup scope. | Customer |
| Which certificate and provisioning-profile workflow is used? | Determines secret inputs and commands. | Customer |
| Is Xcode 14.3 available on the target macOS runner? | Required for compatibility. | Customer / Engineering |
| Must Fastlane or other signing tooling be included? | Changes the runtime and acceptance path. | Customer |

## 9. Validation plan

On the chosen macOS runner, import a non-production signing identity from Harness secrets, unlock the keychain, sign a representative app with Xcode 14.3, verify the signature, and confirm the keychain/profile are removed after success and failure. Review logs for secret material and test runner reuse isolation.

## 10. Dependencies and risks

- Blocking: macOS runner and non-production signing material.
- Planning: pre-provisioned and ephemeral keychains have different controls.
- Implementation: secret leakage and cleanup after cancellation.
- Long-term maintenance: Xcode/macOS image lifecycle and certificate rotation.

## 11. Planning estimate

Qualification only for an existing macOS runner and known signing flow. <1 engineering week for a reusable template/runbook after non-production credentials are available. Runner provisioning is separate.

## 12. What we can tell the customer now

- Xcode keychain operations must run on macOS because Windows has no Apple keychain or codesign service.
- Harness can govern the signing workflow through native steps and secrets.
- We recommend an ephemeral keychain pattern when the runner model allows it.
- The exact keychain, certificate, profile, and Xcode runner setup still needs validation.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 15.

### Bamboo/vendor

- [Apple: Keychain Access User Guide for Mac](https://support.apple.com/guide/keychain-access/kychn001/mac)
- [Apple: What is a certificate?](https://support.apple.com/guide/keychain-access/whats-a-certificate-mchlp2697/mac)
- [Apple Platform Security: Keychain data protection](https://support.apple.com/guide/security/keychain-data-protection-secb0694df1a/web)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/development-guides/mobile/ios.md
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/RunStepInfo.java

Confidence: Medium.
