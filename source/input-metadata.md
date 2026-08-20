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

## Research repository snapshots

The implementation review used the following read-only repository snapshots. Public Bamboo source was used where available; otherwise the active brief links to the official task documentation used to establish behavior.

| Repository | Reviewed revision |
| --- | --- |
| `harness-core` | `4b9442f9229a5f33d300dac097e0a1612c92a3ff` |
| `developer-hub` | `1c7c98f1d76bb7b8330d6ffba96f984878a32748` |
| `harness-community/ci-images` | `9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc` |
| `drone-plugins/drone-artifactory` | `c5db420e97e7c23ce3723aac30deae5b3a714c1e` |
| `harness-community/drone-nunit` | `479806210a6e95b96bc24eefb9f3d41dd953ab4c` |
| `harness-community/drone-cucumber` | `a39f074aa8ee6e77e9f17495ace6dc2ab45fd778` |
| `harness-community/drone-get-maven-version` | `7df46f7c7975996af0ae149ec670f5cbbc65e51a` |
| `harness-community/drone-ant` default branch | `1b6b9eb7a64528c1876c328dfc3bf3c8e500e638` |
| `harness-community/drone-ant` PR 1 head | `53b582d4abfbfb7ffb45561b3d42b7c9f468f310` |
| `DevoKun/bamboo-maven-pom-extractor-plugin` | `83bd81c149de7b2ae562934700cd818347de3c57` |
| `atlassian/bamboo-nodejs-plugin` 9.3.4 release | `9507e81d191890550da1940c175323d220d2418c` |
| `kameshsampath/drone-java-maven-plugin` | `f72fbd12e522cd70d73a1aac58c2c95fa41a57c5` |

No implementation repository was modified as part of this planning work.
