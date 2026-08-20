# Node.js, npm, gulp, grunt, and bower

## Customer need

The customer needs Node-based builds on Windows and lists all versions, with several old npm 5/6 variants elsewhere in the Artifactory tasks. The active Node/npm pairs, package scripts, lockfiles, global versus project-local gulp/grunt/bower, native modules, and registry/cache policy are unknown.

The useful outcome is exact Node selection, predictable package-manager behavior, corporate registry access, cache control, and repeatable project commands. It is not a separate maintained image or plugin for every frontend CLI.

## What Bamboo provides

The last public source-bearing `bamboo-nodejs-plugin` commit reviewed was `9507e81d191890550da1940c175323d220d2418c` from release 9.3.4. Bamboo selects a configured Node executable capability and derives npm from the Node installation. On Windows it executes npm directly; on non-Windows it runs npm through Node. Node and npm tasks accept command/arguments, environment, and working directory. npm can use an isolated temporary cache shared by npm tasks in the build and cleaned afterward.

Gulp, Grunt, and Bower tasks use repository-relative executable paths. Gulp and Grunt take task/config-file inputs. The source expects project-local development dependencies; it does not dynamically download Node.

```text
Bamboo task
-> agent with selected Node capability
-> npm or repository-local tool
-> working directory/environment/cache
-> command result
```

## Harness today

Harness Run steps, templates, secrets, logs, outputs, reports, and Cache Intelligence can govern Node commands. A customer image can supply exact Node/npm. The reviewed `ci-images` repository has no Windows lane. The Drone `npm` plugin is a registry publisher, not a general Node build task. No supported Harness-owned Windows Node build plugin was found.

## Gap

Harness lacks a supported Windows Node selection contract and a clear legacy policy. Telling every pipeline to choose an arbitrary image and type commands loses Bamboo's centralized runtime choice and isolated cache convenience. Baking every historical npm version into images creates an unbounded matrix.

## Recommended approach

Recommendation: use a customer image plus governed package-script template for the POC; productize a Node ecosystem contract with controlled runtime selection and project-local tools.

| Option | Assessment |
| --- | --- |
| Fixed images | Good for a few supported Node LTS versions, but poor for the entire historical set. |
| Broad Java/Node/.NET image | Rejected due to size, conflicts, and unrelated patching. |
| Dynamic Node plugin | Strong fit when it resolves exact verified Node archives and exposes package/cache inputs. |
| Hybrid | Preferred: prepackage current supported versions, resolve other approved versions from a verified cache/mirror, customer images for legacy. |
| Customer image | Fast POC and legacy fallback, but not the full product UX. |

Do not create separate gulp, grunt, or bower plugins. Use package.json scripts or project-local `node_modules/.bin`/`npx` where the project supports it. Bower and global legacy tools remain customer-provided unless active blockers justify qualification.

## POC experience

Proposed template inputs, not final Harness YAML:

```yaml
node:
  version: "20.19.5"
packageManager: npm
command: ci
workingDirectory: web
cache:
  mode: isolated
  dependencyFile: web/package-lock.json
nextCommand: npm test
```

Use the customer's actual pair instead of the example. One POC project verifies package install, project script, registry/proxy/CA, cache behavior, and native-module dependencies.

## Productized direction

Provide a Node-oriented plugin or task contract that calls the shared secure resolver. Inputs should include exact Node version, optional node-version file, package-manager command, working directory, environment, registry auth, proxy/CA/mirror, cache mode/path/key, and project-local executable/script. Node distributions normally bundle npm; support an explicit npm override only when an active legacy pair requires it and the source is approved.

Use Harness-supported current lines, best-effort approved archives, and customer-provided legacy. The resolver must use allowlisted HTTPS or customer mirrors, verified digests, deterministic versions, atomic cache writes, and secret-safe logs.

## Discovery required

- Which exact Node/npm pairs and commands block the POC, and which can be upgraded?
- Are gulp/grunt/bower installed locally, globally, or invoked through package scripts?
- Which lockfiles, registries, proxy/private CA, offline mirror, and cache isolation are required?
- Do native modules require Python, MSBuild, Windows SDK, or other Build Tools?

## Validation

Run representative install/build/test flows on target LTSC. Verify exact Node/npm identity, lockfile enforcement, local tools, scoped registry auth, cache isolation and warm/cold behavior, native modules, paths with spaces, proxy/private CA, offline failure, checksum rejection, cancellation, and secret masking. Compare artifacts and test results with Bamboo.

## Effort and ownership

- POC: included in the 1 to 2 week shared toolchain workstream.
- Product Node contract: 2 to 4 engineering weeks after the shared resolver and inputs are agreed.
- Likely ownership: CI + Platform.

## What we can tell the customer

- Harness can run Node builds on Windows today with a compatible customer image.
- The POC will qualify actual Node/npm pairs and project scripts, not package every historical version.
- The long-term direction provides controlled runtime selection and cache behavior without separate gulp/grunt/bower plugins.
- Native-module projects may also require a selected Visual Studio Build Tools profile.

## Sources

- [Atlassian Node.js and Bamboo guide](https://confluence.atlassian.com/display/BAMBOO0800/Getting%2Bstarted%2Bwith%2BNode.js%2Band%2BBamboo)
- [`bamboo-nodejs-plugin` public source commit `9507e81d191890550da1940c175323d220d2418c`](https://bitbucket.org/atlassian/bamboo-nodejs-plugin/commits/9507e81d191890550da1940c175323d220d2418c)
- [GitHub `setup-node`](https://github.com/actions/setup-node/blob/main/README.md)
- [`drone-npm` purpose](https://github.com/drone-plugins/drone-npm)
- [`ci-images` at `9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc`](https://github.com/harness-community/ci-images/tree/9ffd880e4261a9565b92d8dfc9d45ca8912b0bdc)
