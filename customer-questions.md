# Customer discovery questions

## Questions needed now

These eight questions are the minimum first-call set. Each answer changes the POC scope, architecture, or support commitment.

1. Which capabilities in this plan are POC acceptance blockers, and can you provide one exported Bamboo task configuration plus one representative repository, report, or non-secret fixture for each blocker?

   This prevents qualification work for inactive tasks and gives engineering the actual configured fields rather than a plugin name.

2. Which listed versions must work in the POC, which can move to a supported modern version, and which must remain legacy after migration?

   This separates the POC from a permanent promise for JDK 7, old Node/npm, old Visual Studio, and retired tools.

3. For Java and Node builds, what exact runtime pairs and sources are used: JDK distribution/version, Maven or Ant version, Node/npm version, repository wrappers, custom distributions, and internal mirrors?

   Version alone is not enough to choose a safe runtime source or decide wrapper-first versus tool provisioning.

4. Can POC jobs reach approved public distribution sites, or must all runtimes and dependencies come through an internal Artifactory/mirror with a proxy or private CA?

   This determines whether runtime provisioning is viable and how caches, checksums, and certificates must be designed.

5. Which Visual Studio project types, Build Tools workloads, targeting packs, SDKs, architectures, and `devenv.exe` operations are required?

   The answer decides between a Windows container workload profile and a VM/full-Visual-Studio path.

6. For NUnit and MSTest, which runner executable, framework, adapter, filters/settings, output format, and Test Intelligence outcome are required?

   Visual Studio labels do not identify the runner contract, and current Harness documentation does not confirm Windows C# Test Intelligence versions.

7. Please export the active Artifactory tasks and the POM, Cucumber, and warnings configurations, including wrappers, repositories/file specs, build-info sequencing, POM expressions, result formats/globs, thresholds, and required UI outcome.

   These fields decide whether existing plugin repair, native reporting, a template, or a different product abstraction is needed.

8. For qTest, what product/version, authentication method, project/release/environment mapping, JUnit glob, grouping behavior, and failure policy are used, and is a non-production tenant available?

   A test-tenant API proof is required before committing to a publisher plugin or estimate.

## Evidence package requested once

- Bamboo Specs or configuration exports for the selected POC plans and tasks.
- One sanitized successful log and one expected-failure log per selected capability.
- Representative Java, Node, .NET, Ant, Groovy, POM, NUnit/MSTest, Cucumber, warning, and JUnit fixtures only for selected blockers.
- Target Windows node OS/LTSC version, container isolation mode, and architecture.
- Approved registry, proxy, private CA, mirror endpoints, and non-production JFrog/qTest access.

Secrets, signing keys, production tokens, and proprietary source are not required. Redacted configuration and minimal reproductions are sufficient.

## Questions needed before productization

These should not block the first customer discussion.

### Support and lifecycle

- Which versions will Harness continuously qualify, which are best effort, and which are customer-provided legacy?
- Who approves runtime licenses and distribution sources, especially Oracle or vendor-supported JDK 7?
- What Windows LTSC, architecture, patch cadence, vulnerability response, SBOM, and signing policy will Harness own?
- Which team owns each runtime resolver, image, template, and plugin release?

### Runtime and cache contract

- Which account-level mirrors and custom CA mechanisms should the resolver support?
- Where will verified tool archives be cached, how are entries made immutable, and how are poisoned or revoked entries invalidated?
- Is offline qualification a supported product mode or a customer-specific configuration?

### Ecosystem contracts

- Does Maven require global settings, toolchains, encrypted settings, custom local repository behavior, or publishing configuration beyond the POC?
- Does Node require Corepack, npm overrides, native-module build dependencies, global binaries, or additional package managers?
- Does Ant usage justify a product plugin after the shared Java provider exists?
- Which Groovy scripts are portable, and which Bamboo-bound scripts require redesign?
- Which Build Tools workload profiles merit maintained product images?

### Results and integrations

- Should legacy NUnit/MSTest/Cucumber conversion become one maintained cross-platform normalizer?
- Do warnings require native file/line navigation, baseline comparison, trends, and retention?
- Which qTest API versions, idempotency behavior, rate limits, attachments, object creation, and vendor support policy must the publisher maintain?
