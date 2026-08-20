# Harness Windows CI implementation

## 1. Verified execution models

Bamboo capabilities normally point to JDKs or executables already installed on an agent. A task adds requirements for the selected capability labels, and Bamboo schedules the job only on a matching agent. The task then constructs and runs the command.

Harness Kubernetes uses the image configured in each step:

```text
Run step -> explicit image + command
Plugin step -> explicit plugin image + settings
```

Plugin settings are environment/configuration passed to the selected image. They do not trigger Bamboo-style capability matching and do not replace the configured image with a different runtime image.

## 2. Decision 1: Harness-owned images

### Java build images

Publish one combined Java build family with separate JDK tags:

```text
harness/windows-java-build:temurin8-<ltsc>
harness/windows-java-build:temurin11-<ltsc>
harness/windows-java-build:temurin17-<ltsc>
harness/windows-java-build:temurin21-<ltsc>
```

Each tag contains one JDK, PowerShell/base utilities, one supported Maven fallback, one supported Ant, and one supported Groovy. The Maven, Ant, and Groovy ZIPs reviewed for this plan total about 50 MB compressed, so combining them avoids three image families without putting multiple JDKs in one image. Harness still measures the final extracted Windows image before release.

The latest Temurin Windows x64 JDK ZIPs reviewed for this plan were approximately 106 MB, 199 MB, 191 MB, and 205 MB for Java 8, 11, 17, and 21. Together they are about 702 MB before extraction and before Windows/plugin layers. This supports separate JDK tags rather than one multi-JDK image. JDK 7 requires an explicit support decision.

The standard Maven fallback is preinstalled. A repository wrapper may download its configured Maven distribution when not present in the ephemeral container. That smaller download is allowed only with an approved mirror/checksum; a repeatedly used incompatible Maven version can justify one bounded exception tag.

### Node images

Publish one tag for each customer-required, Harness-approved Node release:

```text
harness/windows-node:<node-version>-<ltsc>
```

Each tag contains one Node runtime and its compatible npm. Do not put all Node versions in one image or publish npm-patch tags without a proven requirement. Native-module profiles can derive from the approved Build Tools layers when needed.

### .NET and Visual Studio images

Publish only confirmed profiles:

```text
harness/windows-dotnet-sdk:<major>-<ltsc>
harness/windows-dotnet-framework:<target>-<ltsc>
harness/windows-vs-buildtools:2022-<workload>-<ltsc>
```

Build Tools, workloads, targeting packs, SDKs, VSTest, and converters are installed during image creation. Workload-specific tags avoid one huge Visual Studio image. Additional VS 2019/2017 or legacy MSBuild profiles require explicit feasibility and support decisions. Full Visual Studio and unchanged `devenv.exe` are unsupported in Windows containers.

### Utility and integration images

- `harness/windows-ci-utility:<ltsc>` for Git and approved small tools.
- Complete Artifactory Plugin tags for Java/Maven/Gradle, Node/npm, and utility download flows.
- Fixed POM Values, warnings parser, conditional Cucumber JSON, and qTest Publisher Plugin images.

Every image is built, signed, scanned, published, patched, tested on Windows Kubernetes, and supported by Harness. Runtime installation and persistent runtime caching are not part of the execution architecture.

## 3. Decision 2: Run versus Plugin

| Capability | Harness step | Harness image strategy | Why |
| --- | --- | --- | --- |
| Maven | Run | Explicit Java build tag for the required JDK | Bamboo mainly adds capability selection and CLI fields; Run plus a template provides execution and reports. |
| Ant | Run | Same explicit Java build tags | Ant task behavior is command construction around the installed executable. |
| Node | Run | Explicit tag for the approved Node release | npm/package scripts are project-native commands; no API integration is added. |
| Groovy | Run | Same explicit Java build tags | Portable scripts need a runtime, while Bamboo-specific APIs must be rewritten. |
| MSBuild | Run | Explicit .NET or workload-specific Build Tools tag | The prepared toolchain image is the core requirement; a plugin would mainly construct the CLI. |
| NUnit | Run | Explicit .NET SDK or NUnit test tag | Run executes tests; prepared conversion tooling plus report paths publishes results. |
| MSTest | Run | Explicit .NET SDK or VSTest/Build Tools test tag | Run executes tests and converts TRX; a plugin adds little POC value. |
| Artifactory | Plugin | Explicit complete Java, Node, or download Plugin tag | Authentication, repository configuration, build-info, transfers, and outputs are integration behavior. |
| Cucumber | Run | Existing language image producing JUnit | Native report ingestion is sufficient; a fixed parser Plugin is conditional only for legacy JSON gates. |
| qTest | Plugin | Explicit fixed Windows qTest Publisher image | API object mapping, submission, polling, retries, and outputs justify a plugin. |

Additional decisions:

- Git and Xcode keychain use governed Run Step Templates.
- POM Values and warnings use fixed Plugin images because they transform data and publish structured outputs.
- Maven dependency orchestration and artifact handoff use native pipeline, trigger, workspace, and repository capabilities rather than build-tool plugins.

## 4. Template and tag usage

A Run Step Template may expose the approved image tag and command parameters as template inputs, but the rendered Run step still contains an explicit `spec.image`. A Plugin template likewise contains an explicit Plugin image tag. This is configuration reuse, not automatic runtime selection.

Use immutable release tags or digests in production templates. Human-friendly tags can point to qualified releases, but pipelines must have a controlled upgrade policy.

## 5. Implementation sequence

1. Windows base-image, LTSC, immutable-tag, signing, SBOM, and release conventions.
2. Java 8/11/17/21 build images and Maven Run template.
3. Ant and Groovy Run templates on the same Java images.
4. Confirmed Node images and Node Run template.
5. Confirmed .NET SDK, .NET Framework, and VS Build Tools workload images.
6. MSBuild, NUnit, and MSTest Run templates plus report converters.
7. Repaired Artifactory code and explicit Java, Node, and download Plugin tags.
8. Native orchestration/artifact templates and selected POM, warnings, Cucumber, and qTest integrations.

## Engineering effort

| Reusable component | Planning estimate |
| --- | --- |
| Windows image/release convention | 1 to 2 engineering weeks |
| Java/Node language-image and Run-template MVP | 1 engineering week after exact versions are confirmed |
| Additional legacy language tag | Estimate after licensing/security review |
| Initial .NET/VS Build Tools profiles and Run templates | 2 to 4 engineering weeks after workloads are known |
| NUnit/MSTest overlays and converters | 1 to 2 engineering weeks after runner versions are known |
| Artifactory repair and Windows Plugin tags | 1 to 2 engineering weeks |
| POM or Cucumber bounded extension | 1 to 2 engineering weeks each if selected |
| Warnings parser | 1 to 2 engineering weeks for confirmed formats |
| qTest Publisher | 2 to 4 engineering weeks after tenant/API proof |

These are planning ranges, not delivery dates. Unconfirmed legacy runtimes and Visual Studio workloads are excluded from the estimates.

## Sources

- [Bamboo capabilities and requirements](https://confluence.atlassian.com/bamboo/about-capabilities-and-requirements-289277171.html)
- [Bamboo Maven task](https://confluence.atlassian.com/bamboo1200/maven-1680480796.html)
- [Harness Run step](https://developer.harness.io/docs/continuous-integration/use-ci/run-step-settings/)
- [Harness Plugin step](https://developer.harness.io/docs/continuous-integration/use-ci/use-drone-plugins/run-a-drone-plugin-in-ci/)
- [Apache Maven Wrapper](https://maven.apache.org/tools/wrapper/index.html)
- [Adoptium API](https://api.adoptium.net/q/swagger-ui/)
- [Microsoft Build Tools in Windows containers](https://learn.microsoft.com/en-us/visualstudio/install/build-tools-container)
