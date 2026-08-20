# Bamboo to Harness CI gap plan

## Why this exists

This planning pack supports a prospective customer's Harness POC and Bamboo migration assessment. It converts the customer's task inventory into one evidence-backed brief per qualifying CI gap. The purpose is to distinguish what can be demonstrated now, what needs Windows/macOS qualification, what requires shared engineering, and what cannot be selected until the customer's actual task configuration is known.

## Scope

A source row is included only when:

1. Harness native equivalent meaningfully contains No or Partial; and
2. Scope classification materially includes CI.

Pure Yes rows and CD/deployment-only or STO/security-only rows are excluded.

- Total source rows: 32
- Included rows: 19
- Excluded rows: 13
- Source and digest: [source/input-metadata.md](source/input-metadata.md)
- Complete selection record: [selection-audit.md](selection-audit.md)

## Executive summary

Primary solution classifications are mutually exclusive:

| Primary planning disposition | Count | Meaning |
| --- | ---: | --- |
| Native capability plus qualification | 4 | NUnit, MSTest, artifact handoff, and macOS keychain/signing; Windows C# TI remains unconfirmed |
| Language/tool image or reusable template | 6 | Maven, Ant, Node, Groovy, Git operations, and Maven dependency orchestration |
| Existing plugin qualification | 0 | No row is qualification-only after source review |
| Existing plugin extension or repair | 3 | Artifactory Maven/Gradle and download need known hardening; npm/build-info is a conditional extension |
| New plugin candidate | 1 | qTest, conditional on P0 discovery, a test-tenant proof, and an ownership decision |
| Discovery-first implementation selection | 5 | MSBuild workloads, warnings, POM extraction, SQL, and Cucumber |
| Product/platform enhancement selected now | 0 | One conditional warnings-UI gap is identified, but customer need is not yet confirmed |

All 19 rows have at least one customer/environment question because the original email supplies inventory and version strings, not exported task configurations. Five rows use discovery as their primary solution type; qTest is additionally discovery-gated before plugin commitment.

## Solution type legend

| Code | Meaning |
| --- | --- |
| A | Existing native Harness capability |
| B | Existing native capability plus qualification |
| C | Language or tool image |
| D | Reusable Harness template |
| E | Existing plugin qualification |
| F | Existing plugin extension or repair |
| G | Discovery-gated new-plugin candidate |
| H | Product/platform enhancement |
| I | Discovery required before selecting implementation |

## Planning table

| Bamboo capability | Customer use | Current state | Gap | Recommended path | Decision gate | Estimate | Confidence | Brief |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Maven | JDK 7/8/11/17/21 estate; exact goals unknown | Governed Windows execution exists when the customer supplies a compatible image | No Harness-maintained Windows Maven/JDK matrix | Shared Windows image plus native Run/Test template | One actual Maven/JDK/LTSC POC pairing | Included in one shared engineering week of image development; not additive | Medium | [Brief](gaps/01-maven.md) |
| Maven dependencies processor | Task present; active relationships unknown | Explicit Harness orchestration exists | Bamboo-derived relationships need explicit migration mapping | Chaining, triggers, ordered stages, and immutable artifact inputs | Supply one active producer-consumer chain | <1 engineering week | Medium | [Brief](gaps/02-maven-dependencies-processor.md) |
| Ant | Task present; versions/targets unknown | Run execution exists; community PR is unqualified | No maintained Windows Ant/JDK environment | Shared image plus Run template | One actual Ant/JDK POC pairing and template acceptance | Included in one shared engineering week of image development; not additive | Medium | [Brief](gaps/03-ant.md) |
| MSBuild/Visual Studio | Versions 2.0 through 17; workloads unknown | Governed Windows execution exists; toolchain is customer-supplied | Build Tools/project compatibility not proven | Workload discovery, then smallest viable image/template or runner path | Representative workloads and projects | Discovery before estimate | Low | [Brief](gaps/04-msbuild-visual-studio.md) |
| NUnit | VS 2015 through customer label “VS 2025”; runner/label unknown | Test/report contracts exist; Windows C# versions are TBD in the current matrix | Windows Test-step, legacy runner, and TI compatibility not proven | Qualify modern Test path; use Run/report fallback where unqualified | Runner/runtime/adapters, TI need, and “VS 2025” meaning | Qualification only; shared legacy work conditional | Medium | [Brief](gaps/05-nunit.md) |
| MSTest | VS 2015 through customer label “VS 2025”; runner/label unknown | Test/report contracts exist; Windows C# versions are TBD in the current matrix | Windows Test-step, VSTest, and TI compatibility not proven | Qualify modern Test path; use Run/report fallback where unqualified | Runner/runtime/adapters, TI need, and “VS 2025” meaning | Qualification only; shared legacy work conditional | Medium | [Brief](gaps/06-mstest.md) |
| Artifact Download | Task present; producer/build selection unknown | Workspace sharing and repository-backed handoff exist | Bamboo selection semantics need mapping | Workspace in-stage; immutable repository handoff across pipelines | Producer/build selection and repository contract | Qualification or <1 engineering week | Medium | [Brief](gaps/07-artifact-download.md) |
| Git tag/commit/push/branch | Operations listed; policy unknown | Native checkout, Run, and scoped credentials exist | No governed mutation template yet | Native checkout plus Git-for-Windows template | Operations, signing, and branch policy | <1 engineering week | Medium | [Brief](gaps/08-git-operations.md) |
| Xcode unlock keychain | Xcode 14.3; signing flow unknown | macOS Run and secret handling exist | No qualified signing template; no Windows equivalent | macOS ephemeral keychain template/runbook | Runner and signing workflow | Qualification or <1 engineering week | Medium | [Brief](gaps/09-xcode-keychain.md) |
| Warnings parser | Task present; formats/UI unknown | Logs, artifacts, outputs, and Markdown annotation primitives exist | No qualified parser/template or structured warning UI | Parser template and summary; platform review only for native file/line UI | Formats, thresholds, and UI outcome | Discovery before estimate | Low | [Brief](gaps/10-warnings-parser.md) |
| Artifactory Maven/Gradle | Tasks present; fields and Gradle versions unknown | Core source and Windows Dockerfiles exist; proxy/validation/output and release gaps remain | Known hardening plus customer qualification | Repair and qualify drone-artifactory | Exported fields, test tenant, output/retry need, image ownership | 1 to 2 weeks, shared | Medium | [Brief](gaps/11-artifactory-maven-gradle.md) |
| Artifactory generic download | Task present; file specs/selectors unknown | Core download source exists; proxy/retry/thread/validation/output and release gaps remain | Known hardening plus customer qualification | Repair and qualify drone-artifactory download | File specs, retry/output need, and test tenant | Included in shared hardening | Medium | [Brief](gaps/12-artifactory-download.md) |
| Artifactory npm/build-info | Legacy npm 5/6 values; operations unknown | Build-info exists; npm command and Node image do not | npm workflow contract missing | Candidate drone-artifactory extension with separate Node variant | Confirm npm operations cannot use bounded vendor-CLI template | 1 to 2 weeks after discovery | Medium | [Brief](gaps/13-artifactory-npm-build-info.md) |
| Maven POM parser | Task present; extracted fields unknown | Version-only source and Windows Dockerfile exist; supported release unverified | Existing plugin emits only project.version | Qualify version-only or extend same plugin | Exact expressions and raw/effective behavior | Discovery before estimate | Low | [Brief](gaps/14-maven-pom-parser.md) |
| Node.js tooling | All versions; exact Node pairs unknown | Governed Windows execution exists when the customer supplies a compatible image | No Harness-maintained Windows Node matrix | Shared Windows image plus native Run templates/cache | One actual Node/npm POC pairing and native-module check | Included in one shared engineering week of image development; not additive | Medium | [Brief](gaps/15-nodejs.md) |
| SQL | Task present; engine and semantics unknown | Governed Run execution exists; no selected client/integration | Database contract unknown | Engine-specific image/template first; plugin only if multi-engine need proven | Engine, auth, scripts, outputs | Discovery before estimate | Low | [Brief](gaps/16-sql-task.md) |
| ScriptRunner Groovy | Generic task present; scripts unknown | Governed Windows execution exists when the customer supplies Groovy/JDK | No maintained image; possible Bamboo API coupling | Direct Groovy Run template; discover coupled scripts | One portable script and Groovy/JDK POC pairing | Included in one shared engineering week of image development; not additive | Medium | [Brief](gaps/17-scriptrunner-groovy.md) |
| Cucumber reports | Task present; format and thresholds unknown | Native JUnit ingestion exists; plugin source is unqualified and defective | Threshold/parser outcome unknown | Native JUnit first; conditionally repair drone-cucumber | Format, gates, Jira behavior | Discovery before estimate | Low | [Brief](gaps/18-cucumber-reports.md) |
| qTest | Task present; API/hierarchy unknown | Harness test reporting exists; no CI-side qTest publisher | qTest synchronization absent | Possible new plugin after API/test-tenant proof | Exported mapping, tenant, and ownership | Discovery before estimate | Low | [Brief](gaps/19-qtest.md) |

## Cross-cutting workstreams

The real planning units are:

1. A one-LTSC Windows image MVP for the finite Java, Node, and Groovy tool matrix.
2. .NET Build Tools workload discovery and shared modern/legacy test qualification.
3. Native pipeline/artifact orchestration and two governed templates for Git and macOS signing.
4. One existing Artifactory plugin qualification/extension workstream.
5. Existing POM and Cucumber plugin decisions after exact field/report discovery.
6. Bounded warnings, SQL, and qTest proofs before implementation selection.
7. Conditional product review only if native structured warning UI is a confirmed POC requirement.

See [cross-cutting-plan.md](cross-cutting-plan.md) for estimates, dependencies, and non-double-counted work.

## What we know now

- Standard Maven, Ant, Node, Git, Groovy, and MSBuild commands do not by themselves justify custom plugins. Harness Run/Test steps provide the managed execution contract around the project or vendor tool.
- harness-community/ci-images is a possible source location, but the reviewed repository has no Windows image lane or established support lifecycle.
- drone-artifactory source implements core Maven, Gradle, download, build-info, direct auth, CA PEM, and Windows Dockerfile paths. Static review found RT proxy bypass, incomplete download retry/thread mapping, weak command validation, and no structured outputs. The workstream is repair plus qualification, and a supported published Windows release was not verified.
- drone-get-maven-version source covers only project.version today; its Windows Dockerfile is not evidence of a supported release.
- drone-cucumber is a useful starting point but has confirmed parsing/failure-semantics defects and no release qualification.
- Harness has native Test/report contracts, but the current C# matrix lists Windows supported versions as TBD; Windows Test/TI and legacy runners require sample-based qualification.
- Xcode keychain operations belong on a macOS runner and have no Windows equivalent.
- qTest is the only current new-plugin candidate because it performs structured result synchronization with an external system.

## Configuration examples and current status

These references show the native configuration model without implying that the proposed Windows images or plugin repairs are already delivered.

| Capability | Example/reference | Current status |
| --- | --- | --- |
| Windows Run step | [Harness Windows CI guide](https://developer.harness.io/docs/continuous-integration/development-guides/ci-windows/) | Available when a compatible customer/project image and Windows infrastructure are supplied |
| Test step and report paths | [Harness Test step configuration](https://developer.harness.io/docs/continuous-integration/use-ci/run-tests/tests-v2/) and [test report reference](https://developer.harness.io/docs/continuous-integration/use-ci/run-tests/test-report-ref/) | Native contracts exist; C# Windows versions and TI remain TBD/qualification |
| Artifact handoff | [Share CI data across steps/stages](https://developer.harness.io/docs/continuous-integration/use-ci/caching-ci-data/share-ci-data-across-steps-and-stages/) and [upload to JFrog](https://developer.harness.io/docs/continuous-integration/use-ci/build-and-upload-artifacts/upload-artifacts/upload-artifacts-to-jfrog/) | Workspace/repository patterns are available; Bamboo selection mapping is customer-specific |
| Git write operations | [Codebase Persist Credentials setting](https://developer.harness.io/docs/continuous-integration/use-ci/codebase-configuration/create-and-configure-a-codebase/) and [GitHub App token pattern](https://developer.harness.io/docs/continuous-integration/secure-ci/github-app-token-in-harness/) | Native credentials/Run primitives exist; governed mutation template is proposed |
| macOS keychain/signing | [Harness iOS CI guide](https://developer.harness.io/docs/continuous-integration/development-guides/mobile/ios/) | macOS commands and secret pattern exist; reusable template and Xcode 14.3 runner need qualification |
| Artifactory Maven/download | [Maven Plugin-step example at the reviewed commit](https://github.com/drone-plugins/drone-artifactory/blob/c5db420e97e7c23ce3723aac30deae5b3a714c1e/docs/MAVEN_README.md) and [download example](https://github.com/drone-plugins/drone-artifactory/blob/c5db420e97e7c23ce3723aac30deae5b3a714c1e/docs/DOWNLOAD_README.md) | Source examples exist; mandatory repair, supported publication, and Windows customer qualification remain |

## What we still need from the customer

The original email confirms the inventory and broad Windows/version context but not exact task configuration. The deduplicated P0/P1 questions and requested evidence package are in [customer-questions.md](customer-questions.md).

## Evidence baseline

- Harness code: harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff
- Harness docs: developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a
- Harness UI searched: harness-core-ui at 23fd2e57e040d7c52598dbb0362ae2f5df4333df
- Existing plugin: drone-artifactory at c5db420e97e7c23ce3723aac30deae5b3a714c1e
- Community sources: ci-images 9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc; drone-cucumber a39f074aa8ee6e77e9f17495ace6dc2ab45fd778; drone-get-maven-version 7df46f7c7975996af0ae149ec670f5cbbc65e51a; drone-ant PR 1 commit 53b582d4abfbfb7ffb45561b3d42b7c9f468f310

No implementation or product repository changes are part of this plan.
# bamboo-harness-ci-gap-plan
