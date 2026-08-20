# Maven POM value extraction

## Customer need

The customer uses the Bamboo Maven POM Value Extractor to expose POM data to later tasks. The actual POM path, GAV versus custom expression, variable name/prefix/scope, SNAPSHOT handling, and raw versus effective-POM requirement are unknown.

The outcome is a deterministic metadata-to-pipeline-output contract on Windows.

## What Bamboo provides

The upstream source at `83bd81c149de7b2ae562934700cd818347de3c57` reads a selected raw POM with Maven's model reader. It can extract groupId/artifactId/version as GAV, strip `-SNAPSHOT`, use the `maven.` or custom prefix, or query one custom JavaBean path such as `properties(source.code.level)` or `dependencies[3].version`. It writes job, result, or plan-scoped Bamboo variables. It does not resolve the effective model; vendor documentation instructs users to generate an effective POM first when inherited/resolved values are needed.

```text
raw or pre-generated effective POM
-> GAV or custom JavaBean property
-> named/scoped Bamboo variable
```

## Harness today

`harness-community/drone-get-maven-version` accepts only a POM directory, invokes Maven `help:evaluate` for `project.version`, and writes only `POM_VERSION`. That uses Maven's evaluated model for one value, not the Bamboo plugin's broader raw-model contract. Its LTSC 2022 Dockerfile installs Maven/OpenJDK through unpinned Chocolatey packages, and no workflows, tests, tags, or release lifecycle were found.

## Gap

The existing utility satisfies only the narrow case “evaluated project.version as one output.” It does not provide GAV, arbitrary JavaBean paths, custom names/prefixes, SNAPSHOT stripping, explicit raw/effective behavior, or a supported Windows release. Extending it without customer fields could choose the wrong semantics.

## Recommended approach

Recommendation: qualify the existing utility only if the customer needs evaluated `project.version`; otherwise replace or extend it with an explicit POM query contract after configuration discovery.

For a broader product path, prefer a small cross-platform parser for raw-model GAV/custom paths and an explicit `model: effective` mode that runs approved Maven with supplied settings when resolution is genuinely required. Do not silently change raw to effective semantics. Reuse the Maven runtime contract instead of installing unpinned packages in this plugin image.

## POC experience

Proposed plugin inputs, not final Harness YAML:

```yaml
pom: services/api/pom.xml
model: raw
queries:
  - expression: groupId
    output: maven_group_id
  - expression: version
    output: maven_version
    stripSnapshot: true
```

If the single required field is evaluated project version, the POC may use the existing plugin after Windows packaging and output qualification.

## Productized direction

Choose one owned repository and define typed query/output behavior, path validation, duplicate output handling, missing/null property policy, XML safety, raw/effective mode, settings/private-parent behavior, and structured Harness outputs. Effective mode shares the Maven resolver, mirror, proxy, CA, and cache controls. Avoid mutating plan-level configuration from a build; pass execution outputs to later stages/pipelines explicitly.

## Discovery required

- Which POM paths, GAV/custom expressions, output names, prefixes, scopes, and SNAPSHOT rules are active?
- Are values expected from the raw POM or an effective POM with parent/profile/property resolution?
- Must outputs cross stages or pipelines, and how are they consumed?

## Validation

Use POMs with parent inheritance, properties, profiles, dependencies, missing fields, namespaces, paths with spaces, and private parents. Verify exact output names/values, SNAPSHOT removal, raw/effective distinction, malformed XML failure, secret masking, and parity with Bamboo for every selected expression.

## Effort and ownership

- Version-only qualification/hardening: less than 1 engineering week.
- Bounded GAV/custom/raw-effective extension: 1 to 2 engineering weeks after discovery.
- Likely ownership: CI; Maven resolver work is shared with the Maven capability.

## What we can tell the customer

- An existing Windows-oriented utility can return evaluated Maven project version, but it does not yet match the full Bamboo extractor.
- Exact fields and raw/effective semantics determine whether qualification or extension is required.
- Product outputs will be explicit Harness execution outputs rather than hidden plan mutation.

## Sources

- [`bamboo-maven-pom-extractor-plugin` at `83bd81c149de7b2ae562934700cd818347de3c57`](https://github.com/DevoKun/bamboo-maven-pom-extractor-plugin/tree/83bd81c149de7b2ae562934700cd818347de3c57)
- [Vendor effective-POM guidance](https://gaptap.atlassian.net/wiki/spaces/BAMMVNEXTR/pages/10715138/Extracting%2Bvalues%2Bfrom%2Bthe%2Beffective%2BPOM)
- [`drone-get-maven-version` at `7df46f7c7975996af0ae149ec670f5cbbc65e51a`](https://github.com/harness-community/drone-get-maven-version/tree/7df46f7c7975996af0ae149ec670f5cbbc65e51a)
