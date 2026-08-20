# Maven POM value extraction

## Customer need

The customer extracts Maven POM values and exposes them to later build tasks. Harness must support the required POM path, GAV or custom expression, output name, SNAPSHOT handling, and raw versus effective-POM behavior.

## How Bamboo handles it

The Bamboo Maven POM Value Extractor reads a selected POM on the agent. It can extract groupId, artifactId, and version, strip `-SNAPSHOT`, or query a custom JavaBean-style path. It writes the value as a job, result, or plan variable.

The plugin reads a raw Maven model. If an effective model is required, the vendor instructs users to generate an effective POM first.

```text
POM file on Bamboo agent
-> GAV or custom property query
-> named Bamboo variable
-> later task consumes value
```

## Harness implementation

Recommendation: replace the version-only community utility with a Harness POM Values plugin built on the Harness Windows Java runtime family.

The existing `drone-get-maven-version` runs Maven `help:evaluate` and emits only `POM_VERSION`. It does not cover GAV, custom paths, output naming, SNAPSHOT removal, or explicit raw/effective behavior.

The Harness plugin should support:

- POM path;
- `raw` or `effective` model mode;
- GAV or a bounded list of custom expressions;
- explicit Harness output names;
- optional SNAPSHOT removal;
- clear missing-property and malformed-POM failure behavior.

```text
Harness POM Values Plugin
-> Harness Windows Java runtime
-> raw POM parser, or supported Maven for effective mode
-> named Harness execution outputs
-> later stage/pipeline consumes outputs
```

Raw mode uses an embedded parser. Effective mode uses the supported Maven installation already present in the selected Java profile, including approved settings and repository access. No Maven or JDK is installed during the pipeline.

## What we still need to confirm

- Which POM paths and GAV/custom expressions are active?
- Are values expected from the raw or effective POM?
- Which output names and SNAPSHOT rules are required?
- Must values cross stages or pipelines?

## Customer position

- Harness will provide POM values as explicit pipeline outputs.
- The supported plugin will cover more than the existing version-only utility.
- It will reuse the Harness Java runtime and will not maintain a separate Java image.

## Sources

- [Bamboo Maven POM extractor source](https://github.com/DevoKun/bamboo-maven-pom-extractor-plugin)
- [Effective-POM guidance](https://gaptap.atlassian.net/wiki/spaces/BAMMVNEXTR/pages/10715138/Extracting%2Bvalues%2Bfrom%2Bthe%2Beffective%2BPOM)
- [`drone-get-maven-version`](https://github.com/harness-community/drone-get-maven-version)
