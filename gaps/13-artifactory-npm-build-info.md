# Artifactory npm and build-info

## Customer need

The customer uses JFrog npm and publish-build-info tasks with npm 5.6.0, 6.4.1, 6.11.3, 6.14.4, 6.14.7, 6.14.8, and other npm 5/6 variants. Harness must provide npm registry resolution/deployment and JFrog build-info on Windows Kubernetes.

The matching Node versions are not listed. npm versions cannot be supported independently from the compatible Node runtime.

## How Bamboo handles it

Bamboo selects a long-lived Windows agent with the configured Node/npm capability already installed. JFrog's npm task configures Artifactory resolution or deployment, runs the npm command, collects dependency/package metadata, and publishes or stages build-info.

```text
Bamboo selects agent with Node + npm
-> JFrog task configures npm repositories and credentials
-> npm command runs
-> build-info published to Artifactory
```

## Harness implementation

Recommendation: add npm build behavior to the existing Harness Artifactory plugin and run it on Harness-maintained Node runtime profiles.

The Harness Node profile contains a supported Node release and its bundled npm before the container starts. The Artifactory plugin configures the JFrog server and npm repositories, runs `npm ci`, `npm run`, or the confirmed publish operation, associates the build name/number, and publishes build-info.

```text
Harness Artifactory npm Plugin
-> Harness Windows Node runtime
-> npm install/run/publish through Artifactory
-> JFrog build-info
-> Harness logs and outputs
```

The user selects a supported Node profile, not an npm image. Harness should not create an image for every npm 5/6 patch version. Exact legacy Node/npm pairs are added only after a security, redistribution, Windows-container, and support review. If the required pair cannot be maintained safely, it is not included in the standard Harness-supported set.

The existing `drone-npm` plugin publishes packages to npm registries but does not provide JFrog build-info, so it does not replace this work.

## What we still need to confirm

- Which Node version is paired with each required npm version?
- Which npm commands, repositories, scopes, and build-info sequence are active?
- Can old npm 5/6 builds move to a currently maintained Node/npm profile?
- Are native modules, proxy/private CA, or offline registries required?

## Customer position

- Harness will extend the existing Artifactory plugin and reuse Harness Node runtime profiles.
- Harness will not maintain an image for each npm patch version.
- Old Node/npm pairs require explicit support decisions before the POC commitment.
- JFrog credentials and registry settings remain governed Harness secrets and plugin inputs.

## Sources

- [JFrog Bamboo Artifactory plugin](https://docs.jfrog.com/integrations/docs/bamboo-artifactory-plug-in)
- [JFrog build-tool setup](https://docs.jfrog.com/artifactory/docs/set-up-a-build-tool-with-jfrog-artifactory)
- [`drone-artifactory`](https://github.com/drone-plugins/drone-artifactory)
- [`drone-npm`](https://github.com/drone-plugins/drone-npm)
