# Customer CI capability plan

## Executive takeaway

The customer can move most of the identified Bamboo CI behavior to Harness without recreating every Bamboo task as a plugin. Harness already supplies the pipeline execution, Windows and macOS runner model, secrets, logs, outputs, test reporting, failure strategies, templates, workspace sharing, and artifact connectors. The missing experience is mainly a supported Windows toolchain contract, qualification against the customer's legacy projects, and a small number of structured external integrations.

The recommended product direction is a hybrid:

- use governed Run or Test templates when the underlying tool is already present and no integration logic is missing;
- use ecosystem plugins when version selection, controlled provisioning, structured inputs, or result processing would otherwise be copied into every pipeline;
- use prebuilt images or VMs for large, OS-coupled toolchains such as Visual Studio Build Tools;
- prefer repository wrappers such as `mvnw.cmd` and `gradlew.bat`;
- let customers supply legacy or licensed runtimes that Harness should not redistribute or promise to maintain.

For the POC, customer-supplied toolchain images are an acceptable acceleration path. Productization requires named owners, supported-version classes, signed releases, runtime provenance, patching, and Windows qualification. SQL execution has been removed from the active CI plan because its documented task contract targets a configured database and fits DB DevOps or CD unless the customer proves it is only CI test-fixture setup.

## Proposed POC workstreams

| Workstream | Customer capabilities | POC direction | Product direction | Confidence |
| --- | --- | --- | --- | --- |
| Windows toolchain foundation | Maven, Ant, Node, portable Groovy | Qualify one customer image and one representative project per required family | Shared, secure runtime bootstrap plus Maven and Node contracts; Ant shares Java; Groovy stays template-first | Medium |
| Windows .NET foundation | MSBuild, Visual Studio, NUnit, MSTest | Inventory workloads, qualify one Build Tools profile, run representative tests, publish native reports | Maintained workload-profile images or VM lane plus a shared cross-platform result-normalization layer where needed | Medium |
| Native orchestration | Maven dependencies processor, artifact download | Demonstrate explicit chaining/triggers and immutable artifact inputs | Versioned templates and documented artifact contract | High |
| Governed platform templates | Git mutation, Xcode keychain | Demonstrate one signed or unsigned Git write flow and one macOS signing flow | Supported templates with credential and cleanup policy | Medium |
| Artifactory integration | Maven, Gradle, generic download, npm/build-info | Repair and qualify the existing plugin on one Windows LTSC baseline and JFrog tenant | Supported release process; share Java/Node runtime strategy instead of multiplying images | Medium |
| Metadata and test-result utilities | POM extraction, Cucumber, warnings | Select the smallest path from customer fixtures; use native JUnit where possible | Extend existing utilities or add a shared normalizer only for confirmed behavior | Medium-low |
| qTest publication | qTest result synchronization | API and test-tenant proof before implementation commitment | Windows-capable publisher plugin if the proof confirms the required mapping contract | Low |

## Capability plan

| Capability | Customer need | Harness direction | Discovery that changes the plan | POC effort | Brief |
| --- | --- | --- | --- | --- | --- |
| Maven | Maven across JDK 7, 8, 11, 17, and 21 on Windows | Customer image for POC; product Maven contract using wrapper-first execution and selectable Java | Active Maven/JDK pairs, distribution, wrapper, mirror/offline policy | 1 to 2 weeks within toolchain POC | [Maven](gaps/01-maven.md) |
| Maven dependency processing | Build ordering from Maven relationships | Explicit stages, pipeline chaining, triggers, and immutable artifact inputs | One active producer-consumer graph | <1 week | [Dependency orchestration](gaps/02-maven-dependencies-processor.md) |
| Ant | Ant targets on Windows Java | Customer image plus template; share the Java provider if productized | JDK, Ant, build file, targets, reports | Included in toolchain POC | [Ant](gaps/03-ant.md) |
| MSBuild and Visual Studio | Projects spanning MSBuild 2.0 to 17 | Prebuilt workload profile or VM, not per-run Visual Studio installation | Project types, workloads, targeting packs, `devenv.exe` need | Discovery, then 1 to 2 weeks for one profile | [MSBuild](gaps/04-msbuild-visual-studio.md) |
| NUnit | Execute NUnit and expose results | Run the actual runner, transform only when required, publish native reports | Runner major, framework, categories, report format, TI need | Qualification; 1 to 2 weeks only for shared legacy work | [NUnit](gaps/05-nunit.md) |
| MSTest | Execute MSTest/VSTest and expose results | Native Test candidate for modern projects; governed VSTest fallback | Runner, adapter, framework, settings, TI need | Qualification; shared with .NET work | [MSTest](gaps/06-mstest.md) |
| Artifact download | Consume producer artifacts | Shared workspace in one stage; immutable repository handoff otherwise | Producer and build-selection semantics | <1 week | [Artifact handoff](gaps/07-artifact-download.md) |
| Git operations | Commit, push, tag, and branch | Native checkout plus governed Git-for-Windows template | Operations, signing, identity, branch policy | <1 week | [Git operations](gaps/08-git-operations.md) |
| Xcode keychain | Make signing identities available to Xcode 14.3 | Ephemeral keychain workflow on a macOS runner | Runner, certificate/profile source, cleanup policy | Qualification or <1 week | [Xcode keychain](gaps/09-xcode-keychain.md) |
| Warnings parser | Parse warnings, gate builds, show findings | File-based parser proof and summary; platform review only for native file navigation/trends | Formats, input source, thresholds, UI outcome | Discovery, then 1 to 2 weeks for bounded formats | [Warnings](gaps/10-warnings-parser.md) |
| Artifactory Maven/Gradle | Build through JFrog and publish build-info | Repair and qualify `drone-artifactory` | JFrog fields, wrappers, Gradle versions, proxy/CA | 1 to 2 weeks, shared | [Artifactory build](gaps/11-artifactory-maven-gradle.md) |
| Artifactory download | Resolve/download with JFrog semantics | Repair and qualify existing download path | File specs, properties, retries, output needs | Included in Artifactory work | [Artifactory download](gaps/12-artifactory-download.md) |
| Artifactory npm/build-info | npm resolution/deploy and build-info | Extend the same plugin only if JFrog CLI mapping is confirmed | npm operations, exact Node/npm pairs, publish sequencing | 1 to 2 weeks after discovery | [Artifactory npm](gaps/13-artifactory-npm-build-info.md) |
| Maven POM values | Turn GAV or custom POM fields into pipeline outputs | Replace or extend the version-only utility after exact query discovery | Raw/effective POM, fields, output names and scope | <1 week version-only; 1 to 2 weeks bounded parity | [POM values](gaps/14-maven-pom-parser.md) |
| Node.js tooling | Node, npm, gulp, grunt, and bower on Windows | Customer image for POC; product Node contract with controlled version selection and project-local tools | Node/npm pairs, global tools, native modules, offline policy | Included in toolchain POC | [Node.js](gaps/15-nodejs.md) |
| ScriptRunner Groovy | Run portable Groovy automation | Direct Groovy execution in a governed template; rewrite Bamboo-coupled scripts | Scripts, Bamboo bindings, Groovy/JDK pair | Included in toolchain POC for one portable script | [Groovy](gaps/17-scriptrunner-groovy.md) |
| Cucumber reports | Publish scenarios and apply any required gates | Native JUnit first; repair `drone-cucumber` only for confirmed JSON thresholds | Format, globs, gates, tags, Jira behavior | Qualification or 1 to 2 weeks bounded repair | [Cucumber](gaps/18-cucumber-reports.md) |
| qTest | Publish JUnit results into qTest hierarchy | Discovery-gated Windows publisher plugin | qTest version, auth, release/environment mapping, tenant | Discovery, then 2 to 4 weeks for bounded plugin | [qTest](gaps/19-qtest.md) |

The row estimates are not additive. Shared work and dependencies are consolidated in [cross-cutting-plan.md](cross-cutting-plan.md).

## Decisions needed now

Harness must name owners for Windows toolchain bootstrap, maintained images, and any community plugin used in the POC. It must also decide the supported-version policy, registry and signing process, vulnerability response, and whether JDK 7 is strictly customer-provided legacy.

The POC should select one Windows LTSC baseline and representative projects before creating an image or plugin matrix. A successful customer image proof demonstrates migration feasibility; it does not by itself create a Harness support commitment.

## Customer questions

Eight questions materially affect the POC. They cover blockers and sample plans, supported versus legacy versions, runtime sources and offline constraints, .NET workloads, test modes, Artifactory, result utilities, and qTest. See [customer-questions.md](customer-questions.md).

## Explicitly outside CI scope

- SQL execution against configured databases is assigned to DB DevOps or CD. It can re-enter the CI plan only for confirmed test-fixture setup or build-time validation against an ephemeral test database. The ownership decision is recorded in [the SQL scope note](out-of-ci-scope/16-sql-task.md).
- UrbanCode Deploy and XebiaLabs XL Deploy or Digital.ai Deploy are deployment products and belong to CD migration planning.
- Veracode, Sonar, and Checkmarx are security and code-quality integrations. Their STO/security work is outside this CI capability plan even when a CI pipeline invokes them.

## Source and research note

There are 18 active capability briefs and 14 excluded source rows. Historical selection and original values are kept only in [selection-audit.md](selection-audit.md) and [source/input-metadata.md](source/input-metadata.md).

Primary local evidence was reviewed at these commits: `harness-core` `4b9442f9229a5f33d300dac097e0a1612c92a3ff`, `developer-hub` `1c7c98f1d76bb7b8330d6ffba96f984878a32748`, `drone-artifactory` `c5db420e97e7c23ce3723aac30deae5b3a714c1e`, `drone-nunit` `479806210a6e95b96bc24eefb9f3d41dd953ab4c`, `drone-cucumber` `a39f074aa8ee6e77e9f17495ace6dc2ab45fd778`, `drone-get-maven-version` `7df46f7c7975996af0ae149ec670f5cbbc65e51a`, `drone-ant` PR 1 `53b582d4abfbfb7ffb45561b3d42b7c9f468f310`, `ci-images` `9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc`, `bamboo-nodejs-plugin` source commit `9507e81d191890550da1940c175323d220d2418c`, and `bamboo-maven-pom-extractor-plugin` `83bd81c149de7b2ae562934700cd818347de3c57`.

No implementation or product repository change is part of this planning pack.
