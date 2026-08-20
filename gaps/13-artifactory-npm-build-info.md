# Artifactory npm and build-info

## Customer need

The customer lists JFrog's Bamboo npm and publish-build-info tasks with npm 5.6.0, 6.4.1, 6.11.3, 6.14.4, 6.14.7, 6.14.8, and other npm 5/6 variants. The matching Node versions, npm operations, Artifactory repository setup, deploy behavior, and build-info publish sequence are unknown.

The customer outcome is authenticated npm resolution/deployment plus correct JFrog build identity and dependency/module metadata.

## What Bamboo provides

JFrog's Bamboo npm task configures npm resolution or deployment through Artifactory, runs the npm operation, and collects build metadata. The publish-build-info task sends accumulated build-info to Artifactory. Bamboo agent capabilities supply the selected Node/npm environment.

```text
Node/npm environment
-> JFrog resolver/deployer configuration
-> npm operation
-> collected build metadata
-> publish build-info
```

Official JFrog docs define the public behavior; the Bamboo implementation source was not located publicly.

## Harness today

`drone-artifactory` implements build-info operations and a JFrog CLI base, but has no npm build command or Node Windows image. Its LTSC 2022 path includes Java/Maven/Gradle, not Node. The separate `drone-npm` plugin found in the Drone index publishes packages to npm registries; it is not a JFrog build-info build integration and does not replace this task.

## Gap

The missing item is the exact npm/JFrog operation contract and a compatible Node/npm runtime. Bundling all old npm versions into the JVM Artifactory image would increase size and conflicts. A separate Node runtime path or shared Node resolver is required if the plugin extension is selected.

## Recommended approach

Recommendation: extend `drone-artifactory` only after an exported task proves that a bounded JFrog CLI npm mapping is required; use the shared Node runtime strategy rather than adding Node to the Maven/Gradle image.

The extension should configure the selected JFrog server and npm repositories, run the exact npm command, collect build-info under explicit build name/number, and optionally publish it. It should preserve lockfile-driven installs, proxy/private CA, and secret-safe auth. Exact Node/npm pair selection belongs to the Node contract. Old npm 5/6 pairs are best effort or customer-provided legacy unless a maintained source and qualification policy is approved.

## POC experience

Proposed plugin inputs, not final Harness YAML:

```yaml
buildTool: npm
command: install
node:
  version: "12.22.12"
npm:
  version: "6.14.16"
url: https://company.jfrog.io/artifactory
credentialsSecret: jfrog-ci
resolveRepository: npm-virtual
deployRepository: npm-local
buildName: web-ci
buildNumber: <+pipeline.sequenceId>
publishBuildInfo: true
```

The versions shown are illustrative only and must be replaced by the customer's actual compatible pair.

## Productized direction

Add one npm command family to the existing plugin, with a Node-oriented Windows release or shared resolver integration. Keep build-info publication in the same codebase. Avoid separate plugins for each npm, gulp, grunt, or bower command. Define supported Node/npm pairs and customer-provided legacy policy.

## Discovery required

- Which npm commands, resolver/deployer repositories, scopes, and publish-build-info sequence are active?
- What exact Node version is paired with each POC npm version, and can it be upgraded?
- Are package-lock files, native modules, global tools, proxy/private CA, or offline mirrors required?
- Which build-info fields or outputs are consumed downstream?

## Validation

Use a non-production JFrog project. Verify exact Node/npm identity, install and publish if active, scoped registry auth, lockfile behavior, native modules if present, proxy/private CA, dependency metadata, modules/artifacts, separate and inline build-info publication, failure/cancellation, and secret masking. Compare JFrog build-info with Bamboo.

## Effort and ownership

- Discovery before commitment.
- Bounded npm/build-info extension and one Windows qualification: 1 to 2 engineering weeks after the shared Node runtime decision.
- Likely ownership: CI + HAR.

## What we can tell the customer

- Existing Artifactory code will be extended rather than duplicated if the npm workflow requires it.
- Node/npm versions will be selected separately from the Java Artifactory image.
- Old npm versions are not automatically a Harness-maintained matrix.
- Exported task fields and exact Node/npm pairs are required before scope is fixed.

## Sources

- [JFrog Bamboo Artifactory plugin](https://docs.jfrog.com/integrations/docs/bamboo-artifactory-plug-in)
- [JFrog build-tool setup](https://docs.jfrog.com/artifactory/docs/set-up-a-build-tool-with-jfrog-artifactory)
- [`drone-artifactory` at `c5db420e97e7c23ce3723aac30deae5b3a714c1e`](https://github.com/drone-plugins/drone-artifactory/tree/c5db420e97e7c23ce3723aac30deae5b3a714c1e)
- [`drone-npm` purpose](https://github.com/drone-plugins/drone-npm)
