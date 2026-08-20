# Node.js, npm, gulp, grunt, and bower

## Customer need

The customer runs Node, npm, gulp, grunt, and bower tasks on Windows and describes the version requirement as “all versions.” Harness needs the actual POC Node/npm pairs before defining the supported set.

The solution must provide the runtime, registry credentials, working directory, environment, dependency-cache settings, commands, and reports in Windows containers.

## How Bamboo handles it

Bamboo administrators install Node versions on long-lived Windows agents and register executable capabilities. Tasks select the Node capability, and Bamboo schedules the job on an agent with that version. npm comes from the selected Node installation.

The Bamboo plugin supplies command, arguments, working directory, environment, optional npm cache isolation, and repository-relative gulp, grunt, or bower execution. It does not provide a remote API integration.

```text
Bamboo task requires Node capability
-> matching Windows agent is selected
-> node/npm or project-local tool executes
```

## Harness implementation

Recommendation: use Harness-maintained Windows Node images through native Run steps. Do not build a Node plugin for the POC.

Harness publishes one explicit tag for each approved Node major or exact legacy pair, for example `harness/windows-node:20-<ltsc>`. Each image contains Node, its bundled npm, PowerShell, certificates/base utilities, and the native-build prerequisites approved for that profile. The pipeline selects the tag directly in the Run step.

```text
Harness Run step
-> explicit harness/windows-node:<approved-major> image
-> npm ci && npm run build
-> Harness logs, outputs, reports, and dependency cache
```

A reusable Run Step Template standardizes registry secrets, command, arguments, workdir, environment, Cache Intelligence path/key, and report paths. Project-local package scripts and `npx` are preferred. Harness does not globally install gulp, grunt, or bower unless an active project proves that requirement.

One image containing every Node release is not proposed. Separate tags make the selected runtime clear and avoid retaining EOL versions and vulnerabilities in every pull. npm normally follows the Node distribution. Exact npm pinning or native modules that need Python/MSBuild require a bounded exception profile after discovery.

## What we still need to confirm

- Which exact Node/npm pairs are hard POC requirements?
- Which commands use package scripts versus global gulp, grunt, or bower?
- Do native modules require Python, MSBuild, or Visual Studio workloads?
- Which registry, proxy/private CA, lockfile, and cache behavior is required?

## Customer position

- Node uses native Run steps with explicit Harness-owned image tags.
- Harness maintains only the Node versions selected for the POC.
- npm follows the selected Node image unless exact pinning is proven necessary.
- No dedicated Node plugin is required.

## Sources

- [Bamboo Node.js guide](https://confluence.atlassian.com/display/BAMBOO0800/Getting%2Bstarted%2Bwith%2BNode.js%2Band%2BBamboo)
- [Bamboo Node plugin source](https://bitbucket.org/atlassian/bamboo-nodejs-plugin/)
- [Harness Run step](https://developer.harness.io/docs/continuous-integration/use-ci/run-step-settings/)
- [Node.js release schedule](https://github.com/nodejs/Release)
- [Harness Cache Intelligence](https://developer.harness.io/docs/continuous-integration/use-ci/caching-ci-data/cache-intelligence/)
