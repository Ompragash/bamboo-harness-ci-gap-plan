# Node.js, npm, gulp, grunt, and bower

| Field | Value |
| --- | --- |
| Bamboo plugin key | bamboo-nodejs-plugin:* |
| Provider | Bamboo native Node.js plugin |
| Customer version(s) | All versions |
| Harness CSV status | No |
| Scope | CI, Windows image and reusable template |
| Recommended Harness approach | Maintained Windows Node image tags with native Run templates and explicit cache paths |
| Solution type | C. Language or tool image |
| Discovery required | Yes |
| Planning confidence | Medium |

## 1. What this Bamboo task does

The Bamboo Node tasks select a configured Node/npm capability and run Node, npm, gulp, grunt, or bower commands. Bamboo adds tool-version selection, working directory, environment, and task outcome around project tooling.

Most modern projects express their build in package.json and install local CLI packages. The build engine remains Node/npm and project scripts.

## 2. How it works in Bamboo

Bamboo job → selected Node/npm capability → npm install/ci or Node/project tool command → package registry and project scripts → exit status and generated reports/artifacts.

Older gulp, grunt, and bower tasks may depend on globally installed CLIs. Current practice is project-local dependencies invoked through npm scripts or npx.

## 3. How the customer uses it

Confirmed customer usage: the inventory says all versions and names npm, node, gulp, grunt, and bower. The Artifactory row lists legacy npm 5/6 releases, but the Node versions and whether tools are global or project-local are absent.

Typical plugin capability: install dependencies, run package scripts, and invoke frontend build tools.

Customer usage context: not confirmed from the available source material.

Smallest question: Which Node versions and commands are active, and are gulp, grunt, and bower declared in package manifests or installed globally?

## 4. What Harness supports today

Harness Run steps can execute Node/npm in a selected image and manage secrets, logs, outputs, reports, timeout, and failure strategy. Cache Intelligence supports Node-related dependency paths, but Windows requires explicit paths and keys.

The reviewed ci-images repository has only Linux Node images, old Node 17/18 content, no Windows lane, and a TBU lifecycle. The CSV says No because a maintained Windows Node image/version strategy is absent.

## 5. The actual gap

The gap is a supported Windows Node image family, version policy, registry/CA/proxy qualification, cache template, and migration guidance for project-local legacy build tools.

“All versions” is not a workable support contract. EOL Node/npm and bower require an explicit migration or time-bounded compatibility decision.

## 6. Recommended Harness solution

Recommendation: publish pinned Windows Node image tags and reusable Run templates for npm ci, npm scripts, and npx-invoked project tools.

The customer selects the image tag, command/script, working directory, registry secret, cache paths, reports, timeout, and failure strategy. Harness owns the execution environment, governance, logs, secrets, outputs, and reusable templates.

Engineering work is pinned Node/npm packaging, checksum/SBOM/rebuild policy, Windows Kubernetes smoke tests, private registry/CA/proxy tests, and cache examples. We should not create separate npm, gulp, grunt, bower, or node plugins, and should not bake global project tools into every image.

Result: repositories use their own package manifests while the platform supplies a maintained Windows runtime.

## 7. Proposed implementation shape

- Base OS: one agreed Windows Server Core LTSC baseline for the POC.
- Tags: LTSC, Node major/minor where required, npm version only when deliberately overridden, image revision.
- Included: Node, npm/corepack if compatible, CA setup helpers; no global gulp/grunt/bower.
- Project-owned: package.json, lockfile, npm scripts, local CLIs, application dependencies.
- Template: npm ci or script, npx tool, workdir, registry secret, cache path/key, report paths, outputs, timeout, and failure strategy.
- Qualification: private registry, Artifactory if used, proxy/CA, spaces, long paths, native modules, cache hit/miss, and failure.

## 8. Discovery needed

| Question | Why it matters | Who can answer |
| --- | --- | --- |
| Which Node/npm pairs appear in active plans? | Defines a finite image matrix. | Customer |
| Are gulp/grunt/bower local dependencies or global capabilities? | Determines migration changes and image contents. | Customer |
| Are native Node modules compiled? | May require Python and MSVC Build Tools. | Customer |
| Which registry, proxy, CA, and cache paths are used? | Required for representative Windows qualification. | Customer |

## 9. Validation plan

Run representative locked installs and project scripts on Windows Kubernetes. Test private registry, scoped packages, proxy/CA, paths with spaces and long paths, local npx tools, cache miss/hit, native-module compilation if present, failed install, secret masking, cancellation, and report/artifact production. Record image digests and runtime versions.

## 10. Dependencies and risks

- Blocking: no finite Node/version/command inventory.
- Planning: “all versions” and bower imply legacy compatibility work.
- Implementation: Windows native modules may pull in the MSBuild workload.
- Long-term maintenance: Node release cadence and EOL rebuild decisions.

## 11. Planning estimate

1 to 2 engineering weeks is the required planning bucket, including qualification contingency. The development portion is included in one shared engineering week for Maven/Ant/Node/Groovy and is not additive by row. This assumes one actual Node/npm pairing and reusable registry/build automation. Legacy versions, native modules, production lifecycle, multi-LTSC, and ARM64 are not included.

## 12. What we can tell the customer now

- Harness supports governed Windows Run execution when a compatible Node image is supplied; Harness does not currently provide the proposed maintained Windows Node image family.
- The planned gap closure is a maintained Windows Node image family and reusable templates.
- Project-local npm scripts and npx are preferred over globally packaged gulp, grunt, or bower.
- We need the actual Node/npm pairs and any native-module requirements before confirming coverage.

## 13. Sources

### Customer

- Original email/table: Fwd: Re: Re: Windows/.NET CI/CD gaps in Harness impacting GBD migration, reviewed 2026-08-20.
- Source inventory row 25.

### Bamboo/vendor

- [Atlassian: Getting started with Node.js and Bamboo](https://confluence.atlassian.com/display/BAMBOO0800/Getting%2Bstarted%2Bwith%2BNode.js%2Band%2BBamboo)
- [GitHub Actions: Building and testing Node.js](https://docs.github.com/en/actions/tutorials/build-and-test-code/nodejs)
- [Bower project notice](https://bower.io/docs/creating-packages/)

### Harness

- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/use-ci/caching-ci-data/cache-intelligence.md
- developer-hub at 2c5df07253e2046f97be4c47e7d323474a612e2a: docs/continuous-integration/development-guides/ci-windows.md
- [harness-community/ci-images at 9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc](https://github.com/harness-community/ci-images/tree/9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc)
- harness-core at 4b9442f9229a5f33d300dac097e0a1612c92a3ff: 879-pipeline-ci-commons/src/main/java/io/harness/beans/steps/stepinfo/RunStepInfo.java

Confidence: Medium.
