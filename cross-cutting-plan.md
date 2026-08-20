# Cross-cutting CI plan

The 19 customer rows do not represent 19 independent engineering projects. This plan groups the shared work and keeps discovery, qualification, implementation, and conditional product work separate.

## Workstreams

| Workstream | Rows covered | Deliverable | Discovery | Estimate | Dependency |
| --- | --- | --- | --- | --- | --- |
| Windows POC image recipes for Java/Node/Groovy | Maven, Ant, Node.js, ScriptRunner Groovy; Maven runtime reused by POM work | One agreed LTSC recipe pattern; one actual customer pairing per Maven/JDK, Ant/JDK, Node, and Groovy/JDK family; smoke fixtures; reusable Run/Test templates | Exact tool versions, LTSC, proxy/CA, EOL policy | One engineering week of development for this single combined POC workstream; use the required 1 to 2 engineering week planning bucket when qualification contingency is included; estimates are not additive by row | Existing registry/build automation, Windows Kubernetes nodes, source owner |
| .NET build and native testing qualification | MSBuild/Visual Studio, NUnit, MSTest | Workload inventory; smallest Build Tools/.NET images; Windows Test-step proof for modern projects; shared VSTest/NUnit Run template and report conversion where Test or legacy modes remain unqualified | Project types, workloads, targeting packs, runner/adapters, TI need | Discovery required before estimate; qualification first because current C# Windows versions are TBD | Representative projects, Microsoft installers, Windows host/image compatibility |
| Build and artifact orchestration | Maven dependencies processor, Artifact Download | Explicit producer-consumer pattern using stages, pipeline chaining, triggers, immutable artifact version inputs, and repository-backed handoff | Active relationship map, latest-success selection, repository/version contract | <1 engineering week for a bounded runbook/template and one representative chain | Artifact repository, pipeline ownership, provenance model |
| Governed Git template | Git operations | Versioned Git-for-Windows mutation template with persisted-credential and separate short-lived-token modes | Git signing/policy and write identity | <1 engineering week for standard operations | SCM write credentials and disposable repository |
| macOS signing runbook | Xcode keychain | Ephemeral keychain/signing template and cleanup runbook on macOS | Keychain/certificate/profile flow | Qualification only when the runner and signing assets exist; <1 engineering week if a reusable template is needed | macOS runner and non-production signing material |
| Warning result experience | Warnings parser | Bounded format parser, outputs, threshold gate, full report artifact, and Harness Markdown annotation | Parser formats and whether native file/line UI is required | Discovery required before estimate; likely 1 to 2 engineering weeks for a few known formats | Harness annotation limits; conditional product/UI contract |
| Existing Artifactory plugin | Artifactory Maven/Gradle, generic download, npm/build-info | Repair RT proxy/validation/output gaps and conditional retry/thread behavior; qualify Maven/Gradle/download; select an npm/build-info extension only after mappings and vendor-CLI fit are known | Exported fields, output/retry need, JFrog version, wrappers, npm operations, Node pairs, test tenant | 2 to 4 engineering weeks for the combined bounded workstream; core repair/qualification is 1 to 2 and a selected npm extension is 1 to 2 | drone-artifactory ownership, published Windows images, JFrog tenant, CA/proxy |
| Maven metadata plugin | Maven POM parser | Qualify drone-get-maven-version for version-only or extend it for confirmed GAV/custom/effective-POM mappings; reuse the Maven runtime without adding image effort | Exact expressions, output names/scopes, raw/effective behavior | Discovery required before estimate; <1 engineering week for version-only hardening or 1 to 2 for bounded extension | Maven image, private parent/settings fixture, repository ownership |
| Cucumber result path | Cucumber reports | Prefer native JUnit ingestion; conditionally repair drone-cucumber for confirmed JSON thresholds/outputs | Format/version, globs, tags, thresholds, Jira behavior | Qualification only for native JUnit; conditional 1 to 2 engineering weeks for core plugin repair | Customer fixtures, plugin ownership, Windows Kubernetes |
| Discovery-gated external integrations | SQL, qTest | Database-specific client image/template when one engine is sufficient; possible qTest API publishing plugin only after a test-tenant proof | Database engine/auth/semantics; qTest API, hierarchy, mappings, attachments | Discovery required before estimate for both; neither is included in the shared language-image estimate | Database test endpoint, driver licensing, qTest tenant/API/service account |

## Shared Windows image strategy

The image work should use logical families, not one oversized container:

| Family | Contents | Reason for separation |
| --- | --- | --- |
| Java build | One JDK line plus Maven and/or Ant variants; Groovy variant can reuse the same base | JDK lifecycle and project wrappers determine compatibility |
| Node | Node/npm only, plus CA/registry support | Node release cadence and legacy npm compatibility differ from Java |
| .NET Build Tools | Exact .NET SDK or selected Visual Studio Build Tools workloads | Very large footprint and Windows/Visual Studio servicing constraints |
| Database client | Only the confirmed vendor client and native auth libraries | Drivers, licensing, and authentication differ by engine |
| Plugin runtime overlays | drone-artifactory JVM or Node variant, small qTest runtime if approved | Integration binaries should not force unrelated toolchains into language images |

harness-community/ci-images is a possible source repository. At reviewed commit 9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc it has Linux Dockerfiles only, no Windows lane, no published release lifecycle, and a README support field marked TBU. Treat repository placement, image ownership, rebuild policy, SBOM/signing, registry, and support lifecycle as deliverables.

The development estimate is one engineering week total for Maven, Ant, Node, and Groovy. Do not sum the four brief estimates. The row briefs use the required 1 to 2 engineering week planning bucket because that bucket also contains qualification contingency. The one-week development estimate assumes existing registry/build automation and one actual customer tool pairing per family. It covers recipes, smoke fixtures, and templates on one LTSC baseline. It does not include production support lifecycle, a broad proxy/CA matrix, SBOM/signing rollout, all listed versions, Visual Studio Build Tools, SQL clients, multiple LTSC releases, or ARM64. Productionization is discovery required before estimate.

## Native Harness experience

Using a Run or Test step does not make the solution an unmanaged script. The step is the managed execution contract:

- the image and version are selected centrally;
- connectors and Harness secrets control credentials;
- templates govern accepted inputs;
- logs, outputs, reports, timeouts, retries, resources, and failure strategies are native configuration;
- RBAC and audit remain in Harness;
- Test steps add native result handling, splitting, and Test Intelligence where the compatibility matrix supports it;
- Cache Intelligence and artifact connectors are applied only for their intended cache and artifact roles.

The command field invokes the project's build engine or the vendor CLI. This is also the normal model for Maven, Node, MSBuild, Git, and Groovy on other CI platforms.

## Existing plugin decisions

| Repository | Reviewed SHA | Decision | Evidence affecting plan |
| --- | --- | --- | --- |
| drone-artifactory | c5db420e97e7c23ce3723aac30deae5b3a714c1e | Repair and qualify core paths; select npm extension only after discovery | Core Maven, Gradle, download, build-info, direct auth, CA PEM, and Windows Dockerfiles exist; RT proxy, download retry/thread, validation, and output gaps are known |
| harness-community/drone-get-maven-version | 7df46f7c7975996af0ae149ec670f5cbbc65e51a | Qualify for version-only; extend after field discovery | Currently emits only POM_VERSION; no test/release pipeline |
| harness-community/drone-cucumber | a39f074aa8ee6e77e9f17495ace6dc2ab45fd778 | Prefer native JUnit; repair this plugin if thresholds/outputs are required | Windows source exists, but targeted tests exposed parser and failure-semantics defects |
| harness-community/drone-ant PR 1 | 53b582d4abfbfb7ffb45561b3d42b7c9f468f310 | Do not make primary solution | Unmerged, goals-only contract, unpinned packages, and a failing checked-in test |

## Conditional product gaps

No row is assigned primary solution type H because the customer outcome is not confirmed enough to select platform work. One case could become product work: native structured file/line/severity warning ingestion, navigation, retention, and UI if summary annotations are not sufficient.

This should not be hidden inside a Windows plugin. It requires a separate product contract and estimate if customer discovery confirms it as a POC blocker.

## Internal P0 decision gates

These are Harness decisions, not questions for the customer.

| Decision | Affects | Why it is P0 |
| --- | --- | --- |
| Name the owning team and repository for Windows images and each community plugin used in the POC. | Images, Artifactory, POM, Cucumber, qTest | Source existence is not a support commitment. |
| Select the image registry, supported tag policy, SBOM/signing requirements, patch/rebuild expectation, and vulnerability response process. | All proposed Windows images | Required before describing an image as maintained or supported. |
| Define EOL and legacy exception policy for JDK 7, old Maven, Node/npm 5/6, old .NET, and Visual Studio workloads. | Java, Node, .NET | Prevents a POC exception from becoming an unbounded matrix. |
| Confirm plugin release ownership, registry publication, test-tenant access, and compatibility policy. | Artifactory, POM, Cucumber, qTest | Required before customer-facing support claims or release qualification. |
| Decide whether reusable Run/Test templates satisfy the product experience for standard tools. | Maven, Ant, MSBuild, Node, Groovy, Git, SQL | Avoids building task wrappers without integration behavior. |

## POC sequence

1. Obtain the P0 exports, representative projects/reports, initial LTSC/architecture, proxy/CA requirements, and test-system access.
2. Demonstrate capabilities available now: governed Run execution, JUnit-compatible reporting, same-stage workspace sharing, and immutable repository handoff.
3. Qualify the Windows C# Test path and create or qualify the proposed Git mutation and macOS signing templates separately.
4. Qualify a representative MSBuild/Build Tools workload before selecting its image or runner path.
5. Build and qualify the smallest one-LTSC image set for the actual Maven/Ant/Node/Groovy projects.
6. Repair and qualify drone-artifactory Maven/Gradle/download before selecting an npm extension.
7. Select version-only versus extension for POM, and native JUnit versus drone-cucumber repair.
8. Run the bounded warning-parser/summary proof before considering any native warning-UI work.
9. Run bounded proofs for SQL and qTest before an implementation commitment.
10. Escalate native warning UI only if the demonstrated summary/threshold outcome does not satisfy the POC.

## Planning conclusion

The credible plan is a small set of shared workstreams, not a Bamboo plugin recreation program. Most rows map to existing Harness execution, testing, reporting, artifact, secret, and template constructs once the correct Windows/macOS environment is available. Artifactory should build on the existing plugin. qTest is the only current new-plugin candidate because it performs structured external-system synchronization. The highest uncertainty remains the customer's exact task configurations and legacy version combinations.
