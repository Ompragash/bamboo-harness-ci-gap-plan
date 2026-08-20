# Artifactory generic resolve and download

## Customer need

The customer uses JFrog's Bamboo generic resolve task to download files from Artifactory into a build workspace. Active patterns/file specs, properties, build/project/module selectors, destination layout, flat behavior, retries, parallelism, and downstream outputs are not known.

The required outcome is authenticated deterministic resolution with JFrog file-spec semantics.

## What Bamboo provides

JFrog's task resolves configured patterns or a file spec against Artifactory, applies repository and property/build selectors, downloads matching files, and reports failure through Bamboo. JFrog's plugin owns authentication and repository semantics around the transfer.

## Harness today

`drone-artifactory` already has a `download` command that accepts an inline spec or spec path plus URL, credentials, build name/number, module, and project. Windows Dockerfiles and PEM CA settings exist.

At the reviewed commit, download command construction does not map the top-level `retries` or `threads` inputs even though those fields exist in the plugin settings. RT command paths also bypass `enable_proxy`, validation can allow empty command/tool success, and no structured download outputs are written.

## Gap

The gap is a hardened, supported Windows release with the customer's exact file-spec behavior, not a new downloader implementation. Retry/thread promises must match code and the selected JFrog CLI contract.

## Recommended approach

Recommendation: repair and qualify the existing `drone-artifactory` download path.

Add strict requirement for exactly one inline spec or spec path, validate command and URL/auth, map confirmed retry/parallel flags, apply proxy consistently, preserve PEM/private CA behavior, return clear no-match/malformed-spec errors, and expose deterministic outputs where the JFrog CLI can provide them. Do not infer a file count by parsing unstable human logs unless the CLI supplies a machine-readable summary.

## POC experience

Proposed plugin inputs, not final Harness YAML:

```yaml
command: download
url: https://company.jfrog.io/artifactory
accessTokenSecret: jfrog-token
specPath: .harness/download-spec.json
buildName: upstream-build
buildNumber: "1042"
project: product
retries: 3
threads: 4
```

The POC uses one customer file spec, a test repository, and the target Windows LTSC node.

## Productized direction

Release the repaired command in the same signed Artifactory plugin image and support matrix as Maven/Gradle. Document exact JFrog CLI flag mappings and whether summary outputs are stable. Keep repository transfer and toolchain runtimes separate.

## Discovery required

- Which inline/file specs, source patterns, properties, build/project/module selectors, and destination rules are active?
- Are retry, parallelism, no-match, checksum, or downstream outputs POC requirements?
- Which JFrog authentication, proxy, CA, and CLI/server versions must be qualified?

## Validation

Verify inline and file specs, one/many/no matches, properties and build selectors, flat/preserve layout, Windows paths and spaces, checksum behavior, retryable and permanent errors, parallel download, malformed JSON, bad credentials, proxy/private CA, cancellation, output accuracy, and secret masking. Compare downloaded paths and content digests with Bamboo.

## Effort and ownership

- Included in the 1 to 2 week Artifactory core repair workstream.
- Likely ownership: CI + HAR.

## What we can tell the customer

- The existing Harness/Drone Artifactory code already implements the core generic download path on Windows.
- Known validation, proxy, retry/thread, output, and release gaps must be fixed before a supported POC.
- One exported file spec is needed to prove equivalent selection and layout.

## Sources

- [JFrog Bamboo Artifactory plugin](https://docs.jfrog.com/integrations/docs/bamboo-artifactory-plug-in)
- [`drone-artifactory` download example](https://github.com/drone-plugins/drone-artifactory/blob/c5db420e97e7c23ce3723aac30deae5b3a714c1e/docs/DOWNLOAD_README.md)
- [`drone-artifactory` source at `c5db420e97e7c23ce3723aac30deae5b3a714c1e`](https://github.com/drone-plugins/drone-artifactory/tree/c5db420e97e7c23ce3723aac30deae5b3a714c1e)
