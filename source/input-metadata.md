# Source input metadata

| Field | Value |
| --- | --- |
| Source file | `/Users/ompragash/Documents/Codex/2026-08-18/oka/outputs/gbd-bamboo-to-harness-windows-customer-plan.csv` |
| SHA256 | `08c6eee8ae9761ed6d2070f1c6c5e231b29a922058c05542896a0272e5e44592` |
| File size | 19,665 bytes |
| Last modified | 2026-08-19 21:49:01 +0530 |
| Header | `Bamboo task (plugin key),Plugin Provider,Version(s) in use,Harness native equivalent,Scope classification,Proposed Harness solution and delivery plan` |
| Data rows | 32 |
| Initial No/Partial CI candidates | 19 |
| Active CI briefs after ownership review | 18 |
| Excluded rows | 14 |

## Selection and parsing

The workspace and `/Users/ompragash/Git` were searched for CSV files containing the exact required header. One matching file was found, so no rows were combined from other versions.

The source was parsed with `Workbook.fromCSV` from `@oai/artifact-tool`. Header shape, row count, and a six-column used range were validated before applying the selection rule.

A row enters the initial candidate set only when:

1. `Harness native equivalent` contains a meaningful `No` or `Partial` classification; and
2. `Scope classification` affirmatively begins with `CI` or `macOS CI` and does not state that CI is outside or excluded from scope.

This prevents phrases such as `excluded from Windows CI scope` from being mistaken for positive CI scope. A second ownership review removed the SQL task from active CI planning because its documented default behavior targets a configured database and fits DB DevOps or CD. Its research note is retained for the conditional ephemeral-test-database case.

## Customer source reference

The CSV was derived from the email thread with subject `Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration`. Only concise, paraphrased customer context will be used in the planning briefs. The email body and recipient information are not copied into this directory.
