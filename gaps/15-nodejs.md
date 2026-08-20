# Node.js, npm, gulp, grunt, and bower

## Customer need

The customer runs Node, npm, gulp, grunt, and bower tasks on Windows and lists broad version coverage. Harness must provide the required Node runtime, package-manager command, working directory, environment, registry credentials, dependency cache settings, and project commands in Windows containers.

The plan must avoid one image per npm version and should not globally package historical frontend tools unless an active project requires them.

## How Bamboo handles it

Bamboo administrators install Node versions on long-lived Windows agents and register executable capabilities. Bamboo schedules the task on an agent with the selected Node capability and derives npm from that Node installation.

The public plugin source shows Node and npm command/argument tasks, working directory and environment handling, an optional isolated npm cache, and repository-relative gulp, grunt, and bower executables. Bamboo expects those frontend tools to be project dependencies rather than installing them for every task.

```text
Bamboo selects Windows agent with Node/npm already installed
-> Node/npm task receives command, arguments, workdir, and environment
-> project-local tools run
-> result returns to Bamboo
```

## Harness implementation

Recommendation: build a Harness Node plugin backed by Harness-maintained Windows Node runtime profiles.

Harness packages only the supported Node major versions required for the POC, with each runtime's bundled npm and the plugin launcher already installed. The user chooses a supported Node profile and command; the Harness abstraction resolves the internal image.

```text
Harness Node Plugin
-> Harness Windows Node profile
-> npm ci / npm run / npx
-> project-local gulp/grunt/bower when required
-> Harness logs, reports, and outputs
```

Proposed inputs: Node profile, npm command or package script, arguments, working directory, environment, registry secret, cache path/key, and report paths. Cache Intelligence can cache npm dependencies using the lockfile; it is not used to install or cache the Node runtime.

Harness should prefer `npm ci`, package.json scripts, and `npx` or `node_modules/.bin`. It should not globally install gulp, grunt, or bower by default. Old Node/npm pairs and globally installed legacy tools require an explicit Harness support decision based on security status, Windows-container compatibility, and redistribution.

## What we still need to confirm

- Which exact Node/npm pairs are hard POC requirements?
- Which commands are package scripts versus global gulp/grunt/bower tasks?
- Do native modules require Python, MSBuild, or a Visual Studio workload?
- Which registry, proxy/private CA, and lockfile/cache behavior is required?

## Customer position

- Harness will provide structured Node execution on Windows Kubernetes.
- Supported Node runtimes will be packaged and maintained by Harness.
- Harness will not create an image for each npm patch version.
- Legacy Node/npm and global tooling require explicit support decisions.

## Sources

- [Atlassian Node.js and Bamboo guide](https://confluence.atlassian.com/display/BAMBOO0800/Getting%2Bstarted%2Bwith%2BNode.js%2Band%2BBamboo)
- [Bamboo Node plugin source](https://bitbucket.org/atlassian/bamboo-nodejs-plugin/)
- [Node.js release schedule](https://github.com/nodejs/Release)
- [Harness Cache Intelligence](https://developer.harness.io/docs/continuous-integration/use-ci/caching-ci-data/cache-intelligence/)
