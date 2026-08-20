# Cross-cutting CI architecture and delivery plan

## Architecture decision

Harness should use a hybrid Windows toolchain model. A fixed image matrix is fast and reproducible but becomes unsustainable across the customer's Java, Node, Visual Studio, and legacy version combinations. A broad image reduces tags but increases image size, attack surface, conflicts, and patching cost. Runtime provisioning provides flexibility but needs a secure cache and mirror design. Customer images accelerate the POC but do not create a consistent supported product.

The chosen split is:

```text
Harness CI primitives
|
+-- Shared Windows toolchain bootstrap
|   +-- Java distributions and versions
|   +-- Maven, Ant, Node, and Groovy archives
|   +-- proxy, CA, mirror, verification, cache, PATH
|
+-- Ecosystem contracts
|   +-- Maven: product candidate
|   +-- Node: product candidate
|   +-- Ant: conditional thin contract, shared Java provider
|   +-- Groovy: template-first, shared Java provider only if needed
|
+-- Heavy immutable environments
|   +-- Visual Studio Build Tools workload profiles
|   +-- VM path for full Visual Studio or unsupported container workloads
|
+-- Test-result layer
|   +-- native Harness test reporting
|   +-- runner-specific transform only where required
|   +-- possible shared side-by-side normalizer after discovery
|
+-- Structured integrations
    +-- existing Artifactory plugin
    +-- conditional Cucumber and POM utilities
    +-- discovery-gated qTest publisher
```

The customer-facing task contract matters more than copying Bamboo's implementation. Bamboo generally selects an executable and JDK from preconfigured agent capabilities. It does not prove that runtime download belongs inside every replacement task. Harness can provide the same useful choice through a plugin, template, image, or customer-supplied runtime according to ecosystem cost.

## Toolchain option comparison

Ratings are qualitative because no customer benchmark or image registry measurement is available.

| Dimension | Fixed images | Broad image | Dynamic plugin | Hybrid | Customer image |
| --- | --- | --- | --- | --- | --- |
| POC speed | High when an image exists | Medium | Medium | High | High |
| Warm startup | High | Low to medium due to pull size | High with a warm tool cache | High for common versions | Depends on customer image |
| Cold startup | High | Low | Low to medium | Medium to high | Depends on registry and size |
| Reproducibility | High with immutable digests | High but conflicts are harder | High only with exact versions and verification | High | Customer-controlled |
| Version breadth | Low to medium | Medium | High | High | High |
| Legacy support | Expensive tag growth | Conflict-prone | Limited by legal/source availability | Customer fallback | High, customer-owned |
| Proxy/offline | High if baked | High if baked | Requires mirror and cache design | High with prepackaged common tools and mirror support | Customer-controlled |
| Security | Small immutable images can be strong | Largest attack surface | Strong only with allowlists, checksums, and cache integrity | Best balance | Customer-controlled |
| Harness image maintenance | High | Medium but costly rebuilds | Low image count, higher bootstrap maintenance | Medium | Low for Harness |
| Customer UX | Simple but many tags | Simple until versions conflict | Best structured selection | Best supported-version experience | Highest customer effort |
| Long-term Harness cost | High across many versions | High servicing and vulnerability load | Medium shared-code cost | Medium | Low, with inconsistent experience |

## Ecosystem decisions

| Ecosystem | Bamboo value beyond the command | POC path | Productized direction | Why |
| --- | --- | --- | --- | --- |
| Maven | Selects Maven and JDK capabilities; supplies POM, goals, environment, working directory, and test results | Customer image plus governed Maven inputs; use `mvnw.cmd` when present | Maven plugin contract with wrapper-first execution, selectable Java, controlled Maven fallback, settings, reports, and outputs | Structured selection is valuable and the Java matrix is large |
| Ant | Selects Ant and JDK capabilities; supplies build file, targets, environment, working directory, and JUnit results | Customer image plus template | Thin Ant contract only if repeated use justifies it; reuse Java bootstrap | Ant invocation is simple, and independent JDK download logic would be wasteful |
| Node | Selects Node capability; derives npm; supports Node/npm and project-relative gulp, grunt, and bower; optional isolated npm cache | Customer image plus package-script template | Node plugin contract with controlled runtime selection, package manager command, cache, and project-local tools | Exact runtime selection and cache behavior justify a contract; separate gulp/grunt/bower plugins do not |
| Groovy | Unknown until scripts are exported; portable scripts need only Groovy/JDK, but scripts may call Bamboo APIs | Customer image plus Run template for one portable script | Keep template-first; share Java bootstrap only for portable scripts | A Groovy plugin cannot preserve Bamboo APIs and adds little for a normal script |
| MSBuild | Selects installed MSBuild or `devenv.exe`; captures project/options/environment and uses an MSBuild response file | Qualify one customer workload profile | Maintained immutable Build Tools profiles or VM lane | Workloads, targeting packs, and licensing dominate; per-run Visual Studio installation is slow and unsafe |

## Shared Windows toolchain bootstrap

The productized Maven and Node contracts, and any later Ant or Groovy contract, should call one bounded internal resolver rather than implement downloading separately. It is not intended to become a general Windows package manager.

A tool request should contain an exact ecosystem, distribution when applicable, version, architecture, and source policy. The resolver returns an installed directory and environment additions. Maven and Gradle wrappers remain repository-owned and are verified according to repository policy.

Required controls:

- allowlisted HTTPS sources and optional account-level customer mirrors;
- deterministic exact-version resolution, with no implicit `latest` in product flows;
- checksums or publisher signatures pinned in trusted metadata;
- no arbitrary remote installer execution by default;
- bounded timeout and retry;
- proxy and private CA support without secret values in logs;
- atomic cache population and verification again on cache restore;
- cache keys that include tool, distribution, version, OS, architecture, and digest;
- a read-only shared cache where possible, with per-run extraction to avoid poisoning;
- offline failure messages that identify the missing approved artifact;
- telemetry for source, cache hit, resolved digest, and duration without credentials.

Tool archives can be stored in an image layer for common supported versions, in a runner-local tool cache for warm self-managed runners, or in Harness cache with integrity checks. Maven and npm dependency caches are different data and use their own lockfile or project-based keys. A runtime cache must never trust only a friendly version string.

## Version support classes

| Class | Harness promise | Typical examples for this customer |
| --- | --- | --- |
| Harness-supported | Harness owns provenance, qualification, patch/rebuild, and documented Windows compatibility | Current supported JDK distributions and Node LTS lines selected after discovery; current Maven/Ant; one or more current Build Tools profiles |
| Compatibility or best effort | Generic mechanism can resolve or run the version, but Harness does not continuously qualify every combination | Less common Maven, Ant, Groovy, or interim Node lines from approved sources |
| Customer-provided legacy | Customer supplies an approved image/archive and accepts EOL/licensing constraints | JDK 7, unsupported Visual Studio workloads, old Node/npm pairs, retired Bower/Scriptrunner dependencies |

Oracle's JDK 7 archive requires an Oracle account and is governed by the archive license. Current public Temurin support does not provide a general JDK 7 answer. Azul advertises older Java support, but access and commercial terms must be confirmed. JDK 7 should therefore be customer-provided legacy or come from a customer-approved vendor mirror, never an unqualified Harness public download.

## Test-result architecture

NUnit, MSTest, and Cucumber separate test execution from report normalization and Harness ingestion.

```text
actual test runner
-> native report where supported
-> optional side-by-side conversion
-> Harness report ingestion and Tests tab
-> optional external publication such as qTest
```

`drone-nunit` is not a runner. It converts existing NUnit XML to JUnit in place and can fail on detected NUnit 3 failures. It is Linux-only, uses CGO/libxml2/libxslt, has no releases, does not correctly count NUnit 2 root failures, and destroys the source report. Porting it unchanged to Windows is not the recommended path.

For the POC, run NUnit or VSTest directly and use their supported result options or a pinned transform. Cucumber should emit JUnit from the test framework where possible. Product work should first extend native formats. If several legacy formats still require conversion, build one cross-platform normalizer that writes a separate JUnit file, preserves the source, emits structured counts, and has shared fixtures. qTest publication remains downstream and separate.

## Delivery workstreams and estimates

| Workstream | POC deliverable | POC effort | Productization deliverable | Productization effort | Likely owner |
| --- | --- | --- | --- | --- | --- |
| Toolchain POC | One LTSC baseline; one Maven/JDK, Ant/JDK, Node, and portable Groovy sample using customer images and governed templates | 1 to 2 weeks total | Shared secure resolver, signed base/release process, compatibility matrix | 2 to 4 weeks for bounded resolver foundation, then ecosystem work | CI + Platform |
| Maven contract | Wrapper-first task experience on one real project | Included in toolchain POC | Maven plugin inputs, Java selection, controlled Maven fallback, reports, outputs | 2 to 4 weeks after field validation | CI |
| Node contract | One exact Node/npm pair and project script | Included in toolchain POC | Node selection, package commands, cache, mirror, project-local tools | 2 to 4 weeks after field validation | CI |
| .NET workload | One representative Build Tools profile and test projects | Discovery, then 1 to 2 weeks | Maintained workload profiles, servicing tests, optional shared normalizer | 2 to 4 weeks per bounded profile or normalizer workstream | CI + Platform |
| Orchestration | One producer-consumer chain and artifact handoff | <1 week | Versioned templates and runbook | <1 week | CI |
| Git and macOS templates | One repository mutation flow; one Xcode 14.3 signing flow | <1 week each after assets exist | Supported templates and credential lifecycle | 1 to 2 weeks shared | CI + Platform |
| Artifactory | Repair known defects and qualify Maven/Gradle/download | 1 to 2 weeks | Release automation, support matrix, optional npm extension | Additional 1 to 2 weeks for selected npm scope | CI + HAR |
| POM/Cucumber/warnings | Fixture-based implementation selection | Qualification or discovery | Bounded utility repair/extension or product warning contract | 1 to 2 weeks per selected bounded utility; warning platform work separate | CI, possibly Platform |
| qTest | Auth, lookup, and submission proof in a test tenant | Discovery required | Publisher plugin with polling, retry, outputs, Windows release | 2 to 4 weeks after proof and ownership | CI + External/vendor |

These are workstream estimates, not row totals. Production ownership, vulnerability response, broad legacy matrices, multiple LTSC releases, and air-gapped qualification can add work.

## POC sequence

1. Obtain the eight P0 answers, exported Bamboo task configurations, and representative projects/reports.
2. Choose one Windows LTSC/architecture and confirm proxy, CA, registry, and mirror constraints.
3. Demonstrate Harness primitives already available: governed Run/Test execution, reports, outputs, templates, shared workspace, connectors, and immutable artifact handoff.
4. Qualify one Java, Node, and .NET project path with customer-provided toolchains. Include one portable Groovy script and Ant project only if they are POC blockers.
5. Qualify NUnit/MSTest report behavior without claiming Windows C# Test Intelligence until the supported matrix and E2E proof agree.
6. Repair and qualify the existing Artifactory plugin core paths.
7. Decide native versus utility paths for POM, Cucumber, and warnings from customer fixtures.
8. Run the qTest test-tenant proof before estimating or promising a publisher.
9. Record which legacy exceptions remain customer-provided and which product work has a named owner.

## Internal decisions before product commitment

- Name repository and on-call owners for the resolver, images, and every community plugin presented as supported.
- Choose distribution allowlists, artifact metadata authority, cache location, signing, SBOM, patch cadence, and vulnerability SLA.
- Set the Harness-supported, best-effort, and customer-provided version lists.
- Decide whether the Maven and Node product contracts are customer-specific POC work or roadmap capabilities.
- Decide whether structured file/line warnings require a native result contract rather than a parser plugin.
- Confirm qTest vendor/API support and tenant ownership before creating a new integration.

## Planning conclusion

The minimum credible POC is a set of qualified Harness-native flows on customer toolchains, not a fleet of new images or plugins. The sustainable product direction is a small shared provisioning foundation, two likely ecosystem contracts, heavy prebuilt .NET profiles, native test reporting, and reuse of the existing Artifactory and community utility code only where its behavior matches the customer need.
