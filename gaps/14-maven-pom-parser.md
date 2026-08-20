# Maven POM parser

| Field | Value |
| --- | --- |
| Bamboo plugin key | maven-pom-parser-plugin |
| Provider | Third party |
| Customer version(s) | Not provided |
| Harness CSV status | No |
| Scope | CI, existing plugin qualification or extension |
| Recommended Harness approach | Qualify drone-get-maven-version if version-only; otherwise extend it after field discovery |
| Solution type | I. Discovery required before selecting implementation |
| Discovery required | Yes |
| Planning confidence | Low |

## 1. What this Bamboo task does

The Bamboo plugin extracts values from a Maven POM and exposes them as Bamboo variables for later tasks. It can supply standard coordinates and custom JavaBean-style paths, rename or prefix outputs, strip SNAPSHOT suffixes, and choose variable scope.

Its value is a deterministic metadata-to-pipeline-output contract.

## 2. How it works in Bamboo

Bamboo job → POM extractor task → raw pom.xml model or separately generated effective POM → configured value paths → Bamboo job/result/plan variables.

Material inputs include POM file, standard or custom expressions, output names/prefix, SNAPSHOT handling, and scope. Vendor documentation says inherited/effective values require generating an effective POM first.

## 3. How the customer uses it

Confirmed customer usage: the inventory includes the plugin but no expressions, output names, prefix, scope, SNAPSHOT option, or raw/effective POM behavior.

Typical plugin capability: expose project version, GAV coordinates, properties, or nested POM model values to later tasks.

Customer usage context: not confirmed from the available source material.

Smallest question: Does the customer extract only project.version, or also GAV/custom paths, prefixes, SNAPSHOT stripping, variable scopes, or effective-POM values?

## 4. What Harness supports today

harness-community/drone-get-maven-version source exists and has an LTSC 2022 Windows Dockerfile. At reviewed commit 7df46f7c7975996af0ae149ec670f5cbbc65e51a it runs Maven help:evaluate for project.version and writes only POM_VERSION.

The repository has no tests, workflows, tags, or releases; its Windows Dockerfile installs unpinned Maven/OpenJDK via Chocolatey. It satisfies only the narrow version-only use case.

The CSV says No because native arbitrary POM-output extraction is absent and the existing plugin's contract may be too narrow.

## 5. The actual gap

If only project.version is needed, the gap is qualification and hardening. If custom values or effective POM semantics are active, the gap is a defined extension: expression-to-output mappings, POM selection, prefix/SNAPSHOT behavior, Maven settings/profiles, and deterministic errors.

## 6. Recommended Harness solution

Recommendation: inspect exported task fields, then use the existing plugin for version-only or extend the same repository for the confirmed broader contract.

The customer configures a Plugin step with POM path, raw/effective mode, output mappings, profiles/settings if required, and SNAPSHOT behavior. Harness manages image, secrets, logs, outputs, timeout, and failure strategy.

Engineering first adds tests, pinned Maven/JDK packaging, Windows paths, wrapper/settings support, and clear malformed/missing-value failure. Broader expressions are added only when confirmed. We should not build a duplicate POM plugin or use a naive XML parser for effective Maven values.

Result: later Harness steps receive named, typed-as-text outputs with traceable Maven semantics.

## 7. Proposed implementation shape

- Existing repository: harness-community/drone-get-maven-version at 7df46f7c7975996af0ae149ec670f5cbbc65e51a.
- Version-only path: keep POM_VERSION, add file-path support, wrapper/settings/profiles, tests, pinned image, and clear failures.
- Extension path: output-name-to-expression mappings, GAV presets, prefix, strip-SNAPSHOT, raw/effective mode, collision validation, output escaping.
- Runtime: reuse the maintained Maven/JDK Windows image because effective POM and inheritance require Maven.
- Fixtures: parent POM, property-derived version, profiles, private parent, malformed POM, missing property, Windows spaces, and output special characters.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which expressions and output names are configured? | Selects qualification versus extension. | Customer |
| Are prefixes, SNAPSHOT stripping, or plan/result scopes used? | Defines output compatibility. | Customer |
| Are inherited/profile values required? | Determines raw versus effective POM and Maven settings. | Customer |
| Are private parents/settings.xml involved? | Requires credentials, CA/proxy, and repository access. | Customer |

## 9. Validation plan

Run the exact exported mappings against representative POMs on Windows Kubernetes. Verify version-only, parent/inherited values, profiles, private parents, missing values, malformed POM, SNAPSHOT handling, output names/escaping, paths with spaces, proxy/CA, and secret masking. Compare output values with Bamboo.

## 10. Dependencies and risks

- Blocking: exported extractor configuration is absent.
- Planning: version-only and arbitrary-model extraction have different scope.
- Implementation: effective POM behavior, private parents, and output escaping.
- Long-term maintenance: unpinned base tools and currently absent release automation.

## 11. Planning estimate

Discovery required before estimate. Version-only hardening may be <1 engineering week. Confirmed GAV/custom/effective-POM extension is likely 1 to 2 engineering weeks. The plugin can reuse the Maven runtime, but this plugin effort is separate from the shared image-recipe estimate.

## 12. What we can tell the customer now

- Maven version plugin source and a Windows Dockerfile already exist, but no supported published Windows release has been verified.
- It currently returns only project.version, so we will not claim broader Bamboo parity without task exports.
- If more fields are required, the existing plugin should be extended instead of duplicated.
- Effective POM values require Maven-aware processing, not simple XML reading.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 24.

### Bamboo/vendor

- [Maven POM Value Extractor overview](https://gaptap.atlassian.net/wiki/spaces/BAMMVNEXTR)
- [Extracting custom variables](https://gaptap.atlassian.net/wiki/spaces/BAMMVNEXTR/pages/10715140/Extracting+custom+variables)
- [Effective POM behavior](https://gaptap.atlassian.net/wiki/spaces/BAMMVNEXTR/pages/10715138/Extracting%2Bvalues%2Bfrom%2Bthe%2Beffective%2BPOM)

### Harness

- [drone-get-maven-version at 7df46f7c7975996af0ae149ec670f5cbbc65e51a](https://github.com/harness-community/drone-get-maven-version/tree/7df46f7c7975996af0ae149ec670f5cbbc65e51a)
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/PluginStepInfo.java

Confidence: Low.
