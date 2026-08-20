# Build warnings

## Customer need

The customer uses Bamboo's warnings parser, but has not supplied parser presets, log/file inputs, encodings, source repositories, thresholds, or the required UI outcome. The need may range from a count and failure gate to clickable file/line findings and historical trends.

The value is structured result processing, not the compiler command itself.

## What Bamboo provides

Bamboo's warnings task parses either prior build logs or matching files, applies a selected parser, can associate warnings with a source repository, classifies severity, applies thresholds, and publishes a warning summary/artifact.

```text
compiler log or report files
-> selected warnings parser
-> normalized findings and severity
-> threshold, summary, artifact, source association
```

Official docs and Specs expose the contract; native Bamboo source was not publicly available.

## Harness today

Harness provides logs, step outputs, artifacts, failure strategies, reusable templates, and bounded Markdown annotations. A file-based parser can run in a governed step and expose counts and a report link. Harness does not currently provide a qualified Bamboo-parser replacement or a confirmed native file/line warning UI with source navigation and trend retention.

## Gap

There are two distinct gaps. Parsing a few known file formats is a bounded utility. Parsing the complete prior step log and presenting native file/line/severity history is a platform result-ingestion feature. A Windows plugin cannot create that UI contract by itself.

## Recommended approach

Recommendation: start with a file-based parser proof for only the customer's active formats, then decide whether summary/threshold output is sufficient.

If the task currently scans build logs, tee the relevant tool output to a file during the POC so the parser has a deterministic input. Emit total/severity counts as outputs, a bounded Markdown summary, and the full normalized report as an artifact. Fail only according to confirmed thresholds. Escalate to a platform enhancement if native source navigation, baselines, or trends are required.

## POC experience

Proposed utility inputs, not final Harness YAML:

```yaml
parser: msbuild
inputFiles: [artifacts/logs/build.log]
sourceRoot: <+codebase.repoUrl>
failThresholds:
  high: 1
  total: 100
summaryLimit: 25
```

This is a plugin only if parser logic and outputs are reused across pipelines. A single stable compiler file format can remain a governed template.

## Productized direction

Maintain a parser utility only for explicitly supported formats and fixtures. If the customer requires clickable file/line findings, new-warning baselines, retention, or trend views, define a native warning schema, ingestion API, repository/file mapping, and UI as a separate Platform product workstream.

## Discovery required

- Which parser presets, log/file sources, globs, encodings, and severities are active?
- Which thresholds change build status?
- Are summary and artifact sufficient, or are file navigation, baseline, and trends POC blockers?

## Validation

Replay sanitized successful and failing inputs. Verify Windows paths, spaces, Unicode, duplicate warnings, malformed data, empty input, severity mapping, thresholds, large reports, summary limits, artifact link, file/repository association where required, and comparison with Bamboo counts.

## Effort and ownership

- Discovery before estimate.
- Bounded parser/template for one to three confirmed formats: 1 to 2 engineering weeks.
- Native warning ingestion/UI: separate product estimate.
- Likely ownership: CI for parsing; CI + Platform for native result experience.

## What we can tell the customer

- Harness can host warning parsing, gates, outputs, artifacts, and a visible summary today.
- The exact parser inputs and desired UI decide whether this is a small utility or platform work.
- We will not reproduce every Bamboo parser preset without an active customer need.

## Sources

- [Atlassian warnings parser task](https://confluence.atlassian.com/bamboo0800/configuring-build-warnings-parser-task-1077778946.html)
- [Bamboo Specs `BuildWarningParserTask`](https://docs.atlassian.com/bamboo-specs/7.2.9/com/atlassian/bamboo/specs/builders/task/BuildWarningParserTask.html)
- [Harness annotations](https://developer.harness.io/docs/platform/pipelines/harness-annotations/)
