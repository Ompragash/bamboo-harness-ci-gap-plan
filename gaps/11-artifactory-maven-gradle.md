# Artifactory Maven and Gradle builds

## Customer need

The customer builds Maven and Gradle projects through JFrog Artifactory and publishes build-info. Harness must preserve repository resolution/deployment, credentials, build name/number, dependency and artifact metadata, proxy/private CA behavior, and build-info publication on Windows Kubernetes.

## How Bamboo handles it

Bamboo selects a Windows agent with Java and Maven or Gradle already installed. JFrog's Bamboo task adds Artifactory resolver/deployer settings, authentication, goals/tasks, build identity, metadata collection, and build-info publication around the installed build tool.

```text
Bamboo selects agent with Java + Maven/Gradle
-> JFrog task configures repositories and credentials
-> build tool runs
-> artifacts/dependencies become Artifactory build-info
```

## Harness implementation

Recommendation: repair and productize the existing `drone-artifactory` codebase as explicitly versioned Windows Plugin images. Artifactory authentication, repository configuration, metadata collection, and build-info publication justify a Plugin step.

The plugin already contains Maven, Gradle, JFrog CLI, build-info, direct authentication, PEM CA, and Windows image paths. The supported version must make these changes:

- reject missing or invalid command/build-tool combinations;
- apply proxy settings consistently to Maven, Gradle, and JFrog CLI paths;
- prefer `mvnw.cmd` and `gradlew.bat` where present;
- map selected build summaries and identifiers to Harness outputs;
- verify downloaded tool binaries during image builds;
- publish signed LTSC images with Windows end-to-end tests and a support owner;
- document that the Plugin-step connector pulls the image, while JFrog credentials come from explicit Harness secrets.

```text
Harness Plugin step
-> explicit harness/drone-artifactory:windows-java17-<tool>-<ltsc> image
-> wrapper or supported build tool
-> JFrog resolver/deployer configuration
-> build-info published to Artifactory
-> Harness logs and outputs
```

Each published tag contains the plugin binary, pinned JFrog CLI, one JDK, and the supported Maven or Gradle fallback for that tag. Repository `mvnw.cmd` or `gradlew.bat` remains preferred where approved. The pipeline chooses the complete image tag explicitly; a plugin setting does not cause Harness to replace it with another Java image.

The POC needs only the JDK and Maven/Gradle combinations actually used. Tags derive from the shared Java image layers, but they are complete plugin images. Harness should not publish variants merely because Dockerfiles exist and should not create a second Artifactory codebase.

## What we still need to confirm

- Which Maven/Gradle/JDK versions and wrappers are active?
- Which resolver/deployer repositories and build-info options are configured?
- Which JFrog version, authentication method, proxy/private CA, and test tenant are available?
- Which Artifactory outputs are consumed later in the pipeline?

## Customer position

- Harness will extend the existing Artifactory integration rather than build a duplicate.
- Complete Plugin images and their Java/toolchain contents will be built, secured, and maintained by Harness.
- Maven/Gradle wrappers are preferred, and JFrog credentials remain Harness secrets.
- The POC targets only the confirmed Windows LTSC and build-tool profiles.

## Sources

- [JFrog Bamboo Artifactory plugin](https://docs.jfrog.com/integrations/docs/bamboo-artifactory-plug-in)
- [JFrog build-info](https://docs.jfrog.com/integrations/docs/about-build-info)
- [`drone-artifactory`](https://github.com/drone-plugins/drone-artifactory)
