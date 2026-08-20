# Harness Windows CI implementation

## 1. Windows plugin and runtime convention

Every language or integration step has two Harness-owned parts:

```text
Harness task/plugin facade
-> validates user inputs
-> selects an internal supported runtime profile
-> Kubernetes starts the Harness image
-> preinstalled tool executes
-> logs, outputs, reports, and failure return to Harness
```

The customer does not select an internal image tag. Harness builds, signs, scans, publishes, tests, patches, and supports the image. Common runtimes are installed during image build, not during CI execution.

Runtime caching is not part of this architecture. Kubernetes Windows containers are ephemeral and can run on different nodes. Cache Intelligence remains available for project dependencies, such as Maven or npm caches, but it does not provide Java, Node, or Visual Studio runtimes.

## 2. Java runtime family

Build four Windows LTSC runtime profiles using pinned Temurin x64 releases:

- Java 8
- Java 11
- Java 17
- Java 21

Each profile shares the same Windows base, plugin launcher convention, certificate handling, health checks, SBOM/signing process, and release tests. Maven, Ant, Groovy, POM, and Java-oriented Artifactory images derive from these profiles.

JDK 7 is outside the standard set. Harness must decide whether an Azul or other supported binary can be redistributed, patched, and tested in the selected Windows container. If that review fails, JDK 7 is unsupported.

## 3. Maven plugin

The Maven plugin selects a Java profile and exposes POM, goals, profiles, arguments, settings secret, `MAVEN_OPTS`, workdir, environment, and reports. It prefers `mvnw.cmd`; each Java profile also includes one Harness-supported Maven fallback.

This creates four Java profiles rather than a Java x Maven image matrix.

## 4. Ant plugin and Groovy wrapper

The Ant plugin uses the same Java profiles and exposes build file, targets, properties, arguments, `ANT_OPTS`, workdir, environment, and JUnit reports. One supported Ant version is included per Java profile unless customer evidence requires another.

The Groovy wrapper also derives from the Java family. It executes portable repository scripts with arguments, environment, secrets, and outputs. Bamboo-coupled scripts are rewritten using Harness pipeline primitives or APIs.

## 5. Node runtime and plugin

Build only the Windows Node major profiles required for the POC and approved for Harness support. Each profile contains Node, its bundled npm, and the plugin launcher.

The Node plugin exposes npm/package-script command, arguments, workdir, environment, registry secret, dependency-cache settings, and report paths. It prefers package.json scripts and project-local `npx` tools. Old npm 5/6 pairs and global gulp/grunt/bower require explicit support decisions.

## 6. Visual Studio and .NET runtime family

Build these initial Harness-owned profiles:

- Windows .NET SDK for modern SDK-style projects;
- Windows .NET Framework SDK for supported full-framework projects;
- Visual Studio Build Tools 2022 for MSBuild 17 and selected container-compatible workloads.

The workload list comes from the customer projects. Build Tools supports Windows containers; full Visual Studio does not. Additional Build Tools 2019/2017 profiles are created only for hard POC blockers. MSBuild 2.0 through 14, `devenv.exe`, installer, SSDT, older SDK, and special C++ workloads need explicit feasibility/support decisions.

## 7. MSBuild plugin

The plugin selects the .NET or Build Tools profile and exposes solution/project, targets, configuration, platform, properties, arguments, response-file behavior, workdir, environment, reports, and artifacts.

The selected Build Tools image already contains the SDKs, targeting packs, and toolsets. No Visual Studio installer runs during the pipeline.

## 8. NUnit and MSTest layer

NUnit and MSTest share the .NET/Build Tools profiles.

- NUnit plugin: executes `dotnet test` or supported NUnit Console, preserves NUnit XML, converts non-destructively when needed, and publishes Harness results.
- MSTest wrapper: executes `dotnet test` or VSTest, collects TRX, and publishes Harness results.

Only verified report transformation behavior should be reused from `drone-nunit`; its Linux-only, destructive implementation is not the production runtime. Windows C# Test Intelligence remains a separate support decision.

## 9. Existing and smaller integrations

- Repair `drone-artifactory`; publish Java, Node, and utility profiles from shared Harness bases.
- Replace the version-only POM utility with a POM Values plugin on the Java family.
- Prefer native JUnit for Cucumber; repair `drone-cucumber` only for required JSON gates.
- Provide Git and warnings on a small Harness Windows utility runtime.
- Build qTest Publisher only after a non-production tenant proves the mapping/API contract.
- Use native pipeline chaining, triggers, workspace sharing, and immutable artifact handoff where no plugin is required.

## 10. Implementation sequence

1. Windows plugin facade, internal profile selection, image signing, and release conventions.
2. Java 8/11/17/21 runtime family.
3. Maven plugin.
4. Ant plugin and Groovy wrapper.
5. Node runtime profiles and Node plugin.
6. .NET SDK, .NET Framework SDK, and VS Build Tools profiles.
7. MSBuild plugin.
8. Shared NUnit/MSTest execution and reporting.
9. Artifactory repair and supported Windows profiles.
10. Native orchestration/templates, then selected POM, Cucumber, warnings, and qTest work.

## Engineering effort

| Reusable component | Planning estimate |
| --- | --- |
| Windows plugin/base/release convention | 1 to 2 engineering weeks |
| Java 8/11/17/21 runtime family | 2 to 4 engineering weeks |
| Maven plugin | 2 to 4 engineering weeks |
| Ant plugin | 1 to 2 engineering weeks |
| Groovy wrapper | 1 to 2 engineering weeks |
| Node runtime profiles and plugin | 2 to 4 engineering weeks after versions are known |
| Initial .NET/VS Build Tools runtime family | 2 to 4 engineering weeks after workloads are known |
| MSBuild plugin | 1 to 2 engineering weeks |
| Shared NUnit/MSTest layer | 2 to 4 engineering weeks |
| Artifactory repair and Windows release | 1 to 2 engineering weeks |
| POM or Cucumber bounded extension | 1 to 2 engineering weeks each if selected |
| Warnings plugin | 1 to 2 engineering weeks for confirmed formats |
| qTest Publisher | 2 to 4 engineering weeks after tenant/API proof |
| Native orchestration and templates | Less than 1 engineering week per representative pattern |

These are component ranges, not dates. Legacy runtimes or additional Visual Studio workload profiles require separate support decisions and estimates.
