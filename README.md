# Bamboo to Harness CI Windows POC

## Correct execution model

```text
Bamboo

preinstalled Windows agents
+ executable/JDK capabilities
+ task requirements select a matching agent
+ Bamboo task invokes the installed tool

                    becomes

Harness Kubernetes Windows

explicit image in each Run or Plugin step
+ required tool already inside that image
+ step command or plugin settings execute in that image
```

Bamboo uses capability matching to choose an agent with the required toolchain. Harness Kubernetes does not perform equivalent toolchain matching for a step. A Run step references an image and command. A Plugin step references a plugin image and settings. In both cases, the configured image is the image that runs; a plugin setting such as `java_version: 17` does not make Harness replace it with another image.

The planning question is therefore which Harness-maintained images are required and whether each capability needs a Plugin abstraction or only a native Run step. Harness builds, publishes, signs, scans, patches, qualifies, and supports every proposed image.

## Final Run versus Plugin model

| Capability | Harness step | Explicit image approach |
| --- | --- | --- |
| Maven | Run | One `windows-java-build` tag per supported JDK |
| Ant | Run | Same Java build image family |
| Groovy | Run | Same Java build image family |
| Node | Run | One Windows Node tag per approved version |
| MSBuild | Run | .NET SDK or workload-specific VS Build Tools tag |
| NUnit | Run | .NET SDK or NUnit/Build Tools test tag |
| MSTest | Run | .NET SDK or VS Build Tools test tag |
| Artifactory | Plugin | Complete Java, Node, or download Plugin image tag |
| Cucumber | Run | Existing language image and native JUnit reporting |
| qTest | Plugin | Fixed Windows qTest Publisher image |

Run is preferred for build tools whose Bamboo task mainly selects an executable and constructs a command. Harness already provides the container lifecycle, secrets, environment, logs, outputs, report paths, timeout, and failure strategy. Reusable Run Step Templates standardize the command and image without maintaining plugin code.

Plugin remains appropriate for integrations that perform API authentication, object resolution, metadata publication, result transformation, or structured outputs.

## Images Harness needs

- `harness/windows-java-build:temurin8|11|17|21` with one JDK plus supported Maven, Ant, and Groovy tooling per tag.
- `harness/windows-node:<approved-version>-<ltsc>` with Node and its compatible npm.
- `harness/windows-dotnet-sdk:<major>-<ltsc>` for modern .NET.
- `harness/windows-dotnet-framework:<target>-<ltsc>` for approved full-framework projects.
- `harness/windows-vs-buildtools:2022-<workload>-<ltsc>` for MSBuild/VSTest workload profiles.
- Test overlays containing approved NUnit/VSTest runners and pinned JUnit conversion tools where the base build image is insufficient.
- Fixed Windows integration images for Artifactory, POM Values, warnings, conditional Cucumber JSON processing, and qTest.
- A small Windows CI utility image with Git and approved base tools.

JDK 7, EOL Node/npm pairs, old MSBuild releases, legacy test runners, and unusual Visual Studio workloads require explicit Harness support decisions. If Harness cannot legally redistribute, secure, patch, and qualify a requested toolchain, it is unsupported rather than transferring maintenance outside Harness.

## POC implementation work

1. Define the Windows base-image, immutable tagging, signing, SBOM, and LTSC compatibility conventions.
2. Build the Java 8/11/17/21 build-image tags and Maven Run template first.
3. Add Ant and Groovy Run templates on the same Java image family.
4. Build only the confirmed Node image tags and Node Run template.
5. Build the confirmed .NET and VS Build Tools workload images and MSBuild Run template.
6. Add NUnit/MSTest test overlays, conversion tooling, and Run templates.
7. Repair and publish explicit Artifactory Plugin image tags.
8. Complete native orchestration/templates and the confirmed POM, Cucumber, warnings, and qTest integrations.

See [cross-cutting-plan.md](cross-cutting-plan.md) for the image and step decisions.

## Customer questions

The eight P0 questions identify the actual image/version matrix, Windows baseline, structured-task UX requirement, Visual Studio workloads, test runners, and integration constraints. See [customer-questions.md](customer-questions.md).

## Capability briefs

| Area | Briefs |
| --- | --- |
| Java | [Maven](gaps/01-maven.md), [dependency orchestration](gaps/02-maven-dependencies-processor.md), [Ant](gaps/03-ant.md), [POM values](gaps/14-maven-pom-parser.md), [Groovy](gaps/17-scriptrunner-groovy.md) |
| .NET and testing | [MSBuild/Visual Studio](gaps/04-msbuild-visual-studio.md), [NUnit](gaps/05-nunit.md), [MSTest](gaps/06-mstest.md), [Cucumber](gaps/18-cucumber-reports.md), [qTest](gaps/19-qtest.md) |
| Artifacts and repositories | [Artifact handoff](gaps/07-artifact-download.md), [Git operations](gaps/08-git-operations.md), [Artifactory Maven/Gradle](gaps/11-artifactory-maven-gradle.md), [Artifactory download](gaps/12-artifactory-download.md), [Artifactory npm/build-info](gaps/13-artifactory-npm-build-info.md) |
| Other CI | [Xcode keychain](gaps/09-xcode-keychain.md), [warnings](gaps/10-warnings-parser.md), [Node](gaps/15-nodejs.md) |

There are 18 active CI capability briefs. Historical selection details remain in [selection-audit.md](selection-audit.md).

## Outside CI scope

- SQL execution against configured databases belongs to DB DevOps or CD. It returns to CI only for a confirmed ephemeral test-database case. See [the SQL scope note](out-of-ci-scope/16-sql-task.md).
- UrbanCode Deploy and XebiaLabs XL Deploy or Digital.ai Deploy belong to CD migration planning.
- Veracode, Sonar, and Checkmarx belong to STO/security planning.

No implementation repository changes are part of this plan.
