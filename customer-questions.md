# Customer questions

These questions are deduplicated across the 19 briefs. Start with the two POC priority questions below. Capability-specific P0 questions become active only for rows the customer marks as POC must-have; this avoids sending the full list as one undifferentiated questionnaire. P1 questions are needed before implementation. P2 questions refine support and maintenance choices.

## POC priority and operating model

| Priority | Question | Affects | Why we need it |
| --- | --- | --- | --- |
| P0 | For each capability, how many active plans use it, is it a POC must-have or deferrable item, which representative plan should be migrated first, and what observable outcome defines acceptance? | All rows | Prevents equal investment in historical or low-value tasks and gives each proof a concrete outcome. |
| P0 | Is a governed, versioned Harness Run/Test template an acceptable native experience for standard build/vendor tools, or is a dedicated Plugin-step form a POC requirement? | Maven, Ant, MSBuild, Node, Groovy, Git, SQL | The answer can change image/template work into a larger plugin-UX request even when the execution behavior is the same. |

## Windows and toolchain

| Priority | Question | Affects | Why we need it |
| --- | --- | --- | --- |
| P0 | Which Windows Server LTSC version and CPU architecture should be the first POC target, and which combinations are later requirements? | All Windows rows | Windows host/image compatibility and binary availability differ across 2019, 2022, 2025, AMD64, and ARM64. |
| P0 | Provide the active Maven, Ant, JDK, Node, npm, Groovy, and wrapper version combinations from the exported plans. | Maven, Ant, Node, Groovy, POM, Artifactory | Replaces “all versions” with a finite image and smoke-test matrix. |
| P0 | Which MSBuild projects are SDK-style .NET, full .NET Framework, C++, installer, database, or devenv-only, and which workloads/targeting packs do they require? | MSBuild, NUnit, MSTest | Determines whether a Windows container is viable and what Build Tools components are required. |
| P1 | Are private package repositories, corporate proxy, custom CA, paths with spaces, long paths, or Windows integrated authentication required? | Images, Artifactory, SQL, tests | These are material Windows acceptance cases. |
| P0 | Do Node projects compile native modules, and are gulp, grunt, or bower installed globally or declared in each repository? | Node, MSBuild | Native modules can add Build Tools and materially change the image and estimate; local package tools avoid global image coupling. |
| P0 | Provide the active Groovy scripts and identify Bamboo API imports, injected objects, Grape dependencies, and required outputs. | ScriptRunner Groovy | Separates direct execution from Bamboo-specific rewrites. |

## Testing

| Priority | Question | Affects | Why we need it |
| --- | --- | --- | --- |
| P0 | Which NUnit/MSTest runner, runtime, adapters, filters, settings files, and representative modern/legacy projects are active? | NUnit, MSTest, MSBuild | Defines native Test/TI versus legacy runner/report qualification. |
| P0 | For Cucumber, provide the plugin version, report format, globs, tag/threshold/fail-on-no-tests settings, and whether Jira actions or a Bamboo report tab are required. | Cucumber | Selects native JUnit ingestion versus bounded plugin repair or separate integration work. |
| P0 | Is Test Intelligence required for the POC, or is complete result ingestion and display sufficient? | NUnit, MSTest | TI has a narrower framework/platform compatibility matrix and changes whether the native path meets the POC outcome. |

## Artifact management

| Priority | Question | Affects | Why we need it |
| --- | --- | --- | --- |
| P0 | Provide one exported Artifactory Maven/Gradle, generic-resolve, and npm/build-info task plus JFrog version, auth model, repositories, and a non-production tenant. | All Artifactory rows | Determines qualification versus exact plugin extension and enables E2E proof. |
| P0 | For Bamboo Artifact Download, identify producer plans, branch/build-selection rules, artifact names, destination, and approved target repository/version contract. | Artifact Download, dependency processor | Separates same-stage workspace sharing from immutable cross-pipeline handoff. |
| P1 | Which build-info project/module, environment, scan, promotion, cleanup, file-spec, no-match, and retry behaviors are required? | Artifactory | Defines acceptance beyond core build and download behavior. |

## SCM

| Priority | Question | Affects | Why we need it |
| --- | --- | --- | --- |
| P0 | Which tag, branch, commit, and push operations/remotes are active, and are signing, LFS, submodules, force-push, or protected-branch exceptions required? | Git operations | Defines the safe reusable template and credential model. |

## Database

| Priority | Question | Affects | Why we need it |
| --- | --- | --- | --- |
| P0 | Which database engines, drivers, authentication modes, script modes, transaction/delimiter behavior, output formats, and DACPAC/migration use are present? | SQL | Selects a database-specific image/template versus a multi-engine integration. |
| P1 | Can required database drivers be redistributed, and is a non-production endpoint available? | SQL | Licensing and E2E access can block implementation. |

## Test management

| Priority | Question | Affects | Why we need it |
| --- | --- | --- | --- |
| P0 | Provide one exported qTest task and confirm qTest version/URL, auth, project/release/build/cycle/environment/grouping mappings, JUnit glob, attachments, object-creation policy, and failure policy. | qTest | Defines the plugin API and result contract. |
| P0 | Is a non-production qTest tenant and least-privilege service account available? | qTest | Required before implementation and E2E qualification of a result publisher. |

## Other CI behavior

| Priority | Question | Affects | Why we need it |
| --- | --- | --- | --- |
| P0 | Does the Maven dependency processor actively create/update cross-plan links, dependency blocking, or transitive scheduling, or is it only present with ordinary Maven dependency resolution? | Maven dependencies processor | Defines which relationships and blocking behavior the explicit Harness orchestration mapping must preserve. |
| P0 | Which POM expressions, output names/prefixes/scopes, SNAPSHOT behavior, profiles, settings, and raw/effective POM semantics are used? | Maven POM parser | Selects existing-plugin qualification versus extension. |
| P0 | Which warning parser formats, globs, thresholds, and UI outcome are required, including file/line navigation or trends? | Warnings parser | Selects a summary template versus platform work. |
| P0 | For Xcode 14.3, is the keychain pre-provisioned or ephemeral, and how are certificates/profiles imported and cleaned up? | Xcode keychain | Defines the macOS template and runner qualification. |
| P0 | Which active producer-consumer chain should be used for the POC demonstration? | Dependency processor, Artifact Download | Provides a bounded orchestration acceptance case. |

## Requested evidence package

The fastest useful response is a redacted export containing:

1. One active configuration for each P0 task family.
2. One representative repository/project or minimal reproduction for each build/test family.
3. One representative Cucumber/warnings/JUnit report file.
4. Initial Windows LTSC and architecture.
5. Non-production JFrog, database, and qTest access where those rows are POC requirements.

Secrets and confidential source code are not required in the planning repository.
