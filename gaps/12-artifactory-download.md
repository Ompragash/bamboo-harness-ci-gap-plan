# Artifactory generic resolve and download

## Customer need

The customer resolves and downloads files from Artifactory into the Windows build workspace. Harness must support the active JFrog file specs or patterns, properties, build/project/module selectors, destination layout, retries, parallelism, private CA, and no-match behavior.

## How Bamboo handles it

JFrog's Bamboo task runs on the selected agent, authenticates to Artifactory, resolves configured patterns or a file spec, downloads matching files into the workspace, and returns the transfer result to Bamboo.

```text
JFrog file spec / selector
-> Bamboo Artifactory task authenticates
-> matching files downloaded to agent workspace
-> success or failure returned to Bamboo
```

## Harness implementation

Recommendation: repair the `download` command in the existing Artifactory plugin and publish one fixed Harness-maintained Windows utility Plugin image.

The current code already accepts an inline spec or spec path, Artifactory URL, credentials, build name/number, project, and module. Harness needs to:

- require exactly one valid inline spec or spec path;
- reject invalid commands, missing URL, or missing authentication clearly;
- apply proxy and private CA settings consistently;
- map confirmed retry and thread inputs to the JFrog CLI;
- handle zero matches and malformed specs according to explicit policy;
- emit stable machine-readable outputs only when the JFrog CLI provides them;
- add Windows LTSC tests, signed image publication, and release ownership.

```text
Harness Plugin step
-> explicit harness/drone-artifactory:windows-download-<ltsc> image
-> validated JFrog file spec
-> JFrog CLI download
-> files in workspace + Harness outputs
```

The Plugin step references that tag directly. File-spec and transfer settings are passed to the running image and never select another image. No Java, Node, or database runtime belongs in this download image. It contains only the plugin launcher, pinned JFrog CLI, required certificates, and standard Windows utility dependencies.

## What we still need to confirm

- Which file specs, patterns, properties, and build/project/module selectors are active?
- What destination, flat/preserve-layout, no-match, retry, and parallel behavior is required?
- Which JFrog version, authentication, proxy/private CA, and test tenant will be used?

## Customer position

- Harness already has the core Artifactory download implementation.
- Harness will repair and maintain the Windows plugin image rather than create another downloader.
- One exported customer file spec is required to prove identical selection and layout.

## Sources

- [JFrog Bamboo Artifactory plugin](https://docs.jfrog.com/integrations/docs/bamboo-artifactory-plug-in)
- [`drone-artifactory` download documentation](https://github.com/drone-plugins/drone-artifactory/blob/master/docs/DOWNLOAD_README.md)
- [`drone-artifactory`](https://github.com/drone-plugins/drone-artifactory)
