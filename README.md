# Bamboo to Harness CI Windows POC

## What is different

```text
Bamboo

Long-lived Windows agent
+ tools installed and registered as capabilities
+ Bamboo task provides structured inputs

                    becomes

Harness Kubernetes Windows

Harness-owned Windows runtime image
+ Harness plugin or governed wrapper
+ one container for each pipeline step
```

Bamboo schedules a job onto an agent where Java, Maven, Node, Visual Studio, or another tool is already installed. Harness Kubernetes schedules a Windows container, so Harness must package the required runtime in a maintained image and provide the task abstraction that invokes it.

Harness should not install large common runtimes during every pipeline run. It should also avoid one image for every tool combination. The recommended model is a small number of Harness-owned runtime families with plugin inputs that hide the internal image profile from the user.

Harness owns the build, publication, security scanning, signing, Windows Kubernetes testing, patching, and support lifecycle for every runtime and plugin image in this plan.

## Proposed runtime model

| Capability | Harness-owned runtime | Harness abstraction |
| --- | --- | --- |
| Maven | Windows Java 8, 11, 17, and 21 profiles | Maven plugin |
| Ant | Same Java runtime family | Ant plugin |
| Groovy | Same Java runtime family with supported Groovy layer | Groovy wrapper |
| Node | Selected supported Windows Node profiles | Node plugin |
| MSBuild | Windows .NET SDK, .NET Framework SDK, and VS Build Tools profiles | MSBuild plugin |
| NUnit | Same .NET/VS test profiles | NUnit plugin with result processing |
| MSTest | Same .NET/VS test profiles | MSTest execution wrapper |
| Git and warnings | Windows utility runtime | Governed Git template and warnings plugin |
| Artifactory | Java, Node, or utility profile according to command | Repaired existing Artifactory plugin |
| POM, Cucumber, qTest | Java or small integration profiles | POM Values, conditional Cucumber Results, and qTest Publisher plugins |

JDK 7, old Node/npm pairs, MSBuild 2.0 through 14, legacy test runners, and unusual Visual Studio workloads are not automatically included. Each requires an explicit Harness decision covering redistribution, security updates, Windows-container feasibility, tests, and support ownership.

Full Visual Studio and `devenv.exe` are not supported in Windows containers. Those tasks must move to Build Tools/MSBuild-compatible commands or be identified as unsupported for this POC environment.

## POC implementation work

1. Define the Harness Windows plugin facade and internal runtime-profile selection convention.
2. Build the Harness Windows Java 8, 11, 17, and 21 runtime family.
3. Build the Maven plugin, then the Ant plugin and Groovy wrapper on that Java family.
4. Build the selected Windows Node runtime profiles and Node plugin.
5. Build the .NET SDK, .NET Framework SDK, and VS Build Tools workload profiles required by the POC.
6. Build the MSBuild plugin and the shared NUnit/MSTest execution and reporting layer.
7. Repair and publish the existing Artifactory plugin for the required Java, Node, and utility profiles.
8. Complete native orchestration/templates, then implement only the confirmed POM, Cucumber, warnings, and qTest work.

The dependency-aware implementation and estimates are in [cross-cutting-plan.md](cross-cutting-plan.md).

## Customer questions

The eight POC questions cover blockers, Windows LTSC, Java/JDK 7, Maven/Ant/Groovy, Node, Visual Studio workloads, test runners, and network/vendor constraints. See [customer-questions.md](customer-questions.md).

## Capability briefs

| Area | Briefs |
| --- | --- |
| Java | [Maven](gaps/01-maven.md), [dependency orchestration](gaps/02-maven-dependencies-processor.md), [Ant](gaps/03-ant.md), [POM values](gaps/14-maven-pom-parser.md), [Groovy](gaps/17-scriptrunner-groovy.md) |
| .NET and testing | [MSBuild/Visual Studio](gaps/04-msbuild-visual-studio.md), [NUnit](gaps/05-nunit.md), [MSTest](gaps/06-mstest.md), [Cucumber](gaps/18-cucumber-reports.md), [qTest](gaps/19-qtest.md) |
| Artifacts and repositories | [Artifact handoff](gaps/07-artifact-download.md), [Git operations](gaps/08-git-operations.md), [Artifactory Maven/Gradle](gaps/11-artifactory-maven-gradle.md), [Artifactory download](gaps/12-artifactory-download.md), [Artifactory npm/build-info](gaps/13-artifactory-npm-build-info.md) |
| Other CI | [Xcode keychain](gaps/09-xcode-keychain.md), [warnings](gaps/10-warnings-parser.md), [Node](gaps/15-nodejs.md) |

There are 18 active CI capability briefs. Historical selection details remain in [selection-audit.md](selection-audit.md).

## Outside CI scope

- SQL execution against configured databases belongs to DB DevOps or CD. It returns to CI scope only for confirmed ephemeral test-database setup or validation. See [the SQL scope note](out-of-ci-scope/16-sql-task.md).
- UrbanCode Deploy and XebiaLabs XL Deploy or Digital.ai Deploy belong to CD migration planning.
- Veracode, Sonar, and Checkmarx belong to STO/security planning.

No implementation repository changes are part of this plan.
