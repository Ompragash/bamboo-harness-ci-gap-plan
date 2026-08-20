# Build warnings

## Customer need

The customer parses compiler or analysis warnings, applies severity thresholds, and makes the findings visible in CI. Harness must support the active warning formats, Windows paths, file inputs or captured logs, failure thresholds, and a useful summary.

## How Bamboo handles it

Bamboo's warnings task runs after the build tool. It reads prior build logs or configured files, applies a selected parser, associates findings with a source repository, classifies severity, applies thresholds, and publishes warning details and a build summary.

```text
compiler log or report files
-> Bamboo warnings parser
-> normalized file/line/severity findings
-> threshold and build summary
```

## Harness implementation

Recommendation: build a small Harness warnings parser plugin and publish it as a fixed Harness-maintained Windows utility image. Parsing, normalization, thresholds, and structured outputs justify a Plugin step.

The first release supports only the customer's active formats. Build commands write or tee relevant output to deterministic files. The plugin reads those files, emits total and severity counts as Harness outputs, fails according to configured thresholds, publishes a bounded Markdown summary, and stores the complete normalized report as an artifact.

```text
build tool writes log/report file
-> Plugin step references harness/warnings-parser:windows-<ltsc>
-> normalized counts + threshold
-> Harness summary and full report artifact
```

Plugin settings are parser type, input file globs, encoding, source root/repository, severity mapping, threshold values, and summary limit. The Plugin step explicitly references the Windows image tag; settings do not select another image. The image contains the parser and required runtime when it starts.

If the customer requires native clickable file/line findings, new-warning baselines, retention, or trend views, that is a separate Harness result-ingestion and UI feature. The parser plugin can produce the data but cannot create a native product experience by itself.

## What we still need to confirm

- Which warning parser presets, input files/logs, globs, and encodings are active?
- Which thresholds change the build result?
- Is a summary plus report artifact sufficient, or are native file links and trends required?

## Customer position

- Harness can provide parsing, thresholds, outputs, summaries, and report artifacts through a fixed Windows Plugin image.
- Harness will support only the formats confirmed for the POC.
- Native file navigation and historical trends require a separate product decision if they are mandatory.

## Sources

- [Atlassian warnings parser task](https://confluence.atlassian.com/bamboo0800/configuring-build-warnings-parser-task-1077778946.html)
- [Bamboo Specs warnings task](https://docs.atlassian.com/bamboo-specs/7.2.9/com/atlassian/bamboo/specs/builders/task/BuildWarningParserTask.html)
- [Harness annotations](https://developer.harness.io/docs/platform/pipelines/harness-annotations/)
