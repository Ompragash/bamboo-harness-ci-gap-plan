# Selection audit

Source: see [source/input-metadata.md](source/input-metadata.md).

Initial filter: select rows whose Harness classification contains `No` or `Partial` and whose supplied scope affirmatively includes CI. Then apply a product-ownership review so command executability alone does not turn a DB DevOps, CD, or STO capability into a CI commitment. Purely `Yes` rows and CD-only, deployment-only, STO/security-only, and database-operation-only rows are excluded.

| Row | Bamboo task | Harness classification | Scope | Included? | Reason |
| --- | --- | --- | --- | --- | --- |
| 2 | Source Code Checkout | Yes - built-in Clone codebase / Git Clone | CI - existing native capability | No | Harness classification is purely Yes. |
| 3 | Maven | No - Run step with a Maven image; Run Tests for tests | CI - Windows image and reusable template | Yes | No/Partial and CI is materially in scope. |
| 4 | Maven dependencies processor | No | CI - pipeline orchestration | Yes | No/Partial and CI is materially in scope. |
| 5 | Ant | No - Run step | CI - Windows image and reusable template | Yes | No/Partial and CI is materially in scope. |
| 6 | MSBuild / Visual Studio | No - Run step on Windows/.NET image | CI - Windows image/template; workload discovery | Yes | No/Partial and CI is materially in scope. |
| 7 | NUnit | Partial - Run step plus JUnit-path reporting | CI - native Test/TI; legacy-mode discovery | Yes | No/Partial and CI is materially in scope. |
| 8 | MSTest runner | Partial - Run step plus JUnit-path reporting | CI - native Test/TI; legacy-mode discovery | Yes | No/Partial and CI is materially in scope. |
| 9 | JUnit / TestNG parser | Yes - report paths | CI - existing native test reporting | No | Harness classification is purely Yes. |
| 10 | Clean working directory | Yes - fresh workspace/pod | CI - existing native infrastructure behavior | No | Harness classification is purely Yes. |
| 11 | Artifact Download | Partial - workspace/cache/repository paths differ | CI/CD - artifact handoff | Yes | No/Partial and CI is materially in scope. |
| 12 | SSH task | Yes - CD Command on SSH/WinRM | CD / deployment-specific - existing native capability | No | Harness classification is purely Yes and scope is CD. |
| 13 | SCP task | Yes - CD Command Copy | CD / deployment-specific - existing native capability | No | Harness classification is purely Yes and scope is CD. |
| 14 | Repository tag / commit / push / branch | No - Run/Shell with Git or plugin | CI - governed reusable template | Yes | No/Partial and CI is materially in scope. |
| 15 | Xcode unlock keychain | No - macOS Run step | macOS CI - outside Windows scope | Yes | CI is materially in scope even though Windows is not. |
| 16 | Warnings parser | No | CI - discovery / conditional plugin | Yes | No/Partial and CI is materially in scope. |
| 17 | Tomcat deploy application | No - CD Command/Shell | CD / deployment-specific - discovery | No | Scope is outside CI. |
| 18 | Agent requirement task | Yes - delegate/infrastructure selectors | CI - existing native infrastructure selection | No | Harness classification is purely Yes. |
| 19 | Sonar for Bamboo | Mixed: Linux Yes / Windows No | STO / security - Windows discovery | No | Scope is STO/security only for this plan. |
| 20 | Artifactory Maven / Gradle build | No - Run step | CI - existing plugin extension | Yes | No/Partial and CI is materially in scope. |
| 21 | Artifactory generic deploy/upload | Yes - native upload step | CI - existing native artifact step | No | Harness classification is purely Yes. |
| 22 | Artifactory generic resolve/download | No - JFrog CLI/curl | CI - existing plugin qualification | Yes | No/Partial and CI is materially in scope. |
| 23 | Artifactory npm plus build-info | No - npm/JFrog CLI | CI - existing plugin extension | Yes | No/Partial and CI is materially in scope. |
| 24 | Maven POM parser | No - read POM into outputs | CI - existing plugin qualification/extension | Yes | No/Partial and CI is materially in scope. |
| 25 | Node.js npm/node/gulp/grunt/bower | No - Node image plus Cache Intelligence | CI - Windows image and reusable template | Yes | No/Partial and CI is materially in scope. |
| 26 | Checkmarx | Mixed: Linux STO Yes / Windows No | STO / security - Windows discovery | No | Scope is STO/security only for this plan. |
| 27 | Veracode | Mixed: STO Yes / Windows No | STO / security - Windows discovery | No | Scope is STO/security only for this plan. |
| 28 | IBM UrbanCode Deploy | No - re-platform to Harness CD | CD / deployment-specific - excluded from Windows CI | No | Scope explicitly excludes CI. |
| 29 | XebiaLabs XL Deploy / Digital.ai Deploy | No - re-platform to Harness CD | CD / deployment-specific - excluded from Windows CI | No | Scope explicitly excludes CI. |
| 30 | SQL task | No - Run step with DB client | Initially labeled CI - discovery / database tool image | No | Ownership review found that the documented task executes against a configured JDBC database. It belongs to DB DevOps or CD unless customer evidence shows an ephemeral CI test-fixture use. Research is preserved in `out-of-ci-scope/16-sql-task.md`. |
| 31 | ScriptRunner Groovy | No - Run/Plugin step | CI - Windows image and reusable template | Yes | No/Partial and CI is materially in scope. |
| 32 | Cucumber reports | No - Run/Plugin step | CI - native reporting or existing plugin qualification | Yes | No/Partial and CI is materially in scope. |
| 33 | qTest | No - Run/Plugin step | CI / test management - discovery-gated plugin | Yes | No/Partial and CI is materially in scope. |

## Result

- Total source rows: 32
- Initial filter candidates: 19
- Active CI briefs after ownership review: 18
- Excluded: 14
- Out-of-CI research notes: 1
