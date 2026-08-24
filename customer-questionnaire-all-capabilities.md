# Bamboo to Harness Windows POC Customer Questionnaire

The customer does not need to manually reproduce information already contained in Bamboo exports or project files. Harness will use the supplied evidence to derive commands, arguments, working directories, report paths, image contents, and migration templates.

Please answer only the sections selected for the POC.

## Information needed once for the POC

1. Which capabilities are hard POC acceptance blockers?
2. What Windows Server LTSC version, container isolation mode, and CPU architecture will the Kubernetes workers use?
3. Can builds access approved public download sources, or must all downloads use internal mirrors, proxies, or private certificate authorities?
4. For each selected capability, please provide:
   - the exported Bamboo plan/job and task configuration;
   - the Bamboo capability label, executable path, and actual installed version;
   - a minimal representative project, relevant project/configuration files, or sample report;
   - one sanitized successful log and, where useful, one expected-failure log.

Harness will inspect this material and determine the required command, image, template, plugin settings, outputs, and result handling.

## Maven (`plugins.maven:task.builder.mvn`)

1. Which exact JDK and Maven versions must be supported for the POC?
2. Which Java distributions are required?
3. Is JDK 7 a hard POC blocker?
4. Please include the relevant `pom.xml`, `.mvn` directory, and `mvnw.cmd` where present.
5. Is an approved Harness Run Step Template acceptable, or is reproducing Bamboo's structured Maven form a hard acceptance requirement?

## Maven Dependencies Processor (`plugins.maven:task.mvn.dependencies.processor`)

1. Which generated producer-consumer plan relationships are POC blockers?
2. Please provide the Bamboo plan dependency configuration or an export/diagram of those relationships.
3. Is preserving “latest successful build” selection mandatory, or can consumers receive an explicit artifact version and digest?

Harness will derive branch matching, trigger ordering, blocking, and fan-out/fan-in behavior from the supplied configuration.

## Ant (`plugins.ant:task.builder.ant`)

1. Which exact JDK and Ant versions must be supported?
2. Please include the relevant `build.xml` files and any custom Ant distributions, libraries, or third-party tasks.
3. Is an approved Harness Run Step Template acceptable, or is reproducing Bamboo's structured Ant form a hard acceptance requirement?

Harness will derive targets, properties, `ANT_OPTS`, arguments, working directories, and report paths from the Bamboo export.

## MSBuild and Visual Studio (`plugin.dotnet:msbuild`, `plugin.dotnet:devenv`)

1. Which exact MSBuild, Visual Studio, or Visual Studio Build Tools versions and executable paths must be supported?
2. Please provide the exported Bamboo tasks and representative solution/project files, such as `.sln`, `.csproj`, `.vcxproj`, `.sqlproj`, or `.vdproj` files.
3. Please provide the installed Visual Studio workload/component inventory, preferably from `vswhere`.
4. Are any `devenv.exe` tasks hard POC blockers?

Harness will derive solution/project commands, targets, configuration, platform, SDKs, targeting packs, workloads, reports, and artifacts from this evidence.

## NUnit (`plugin.dotnet:nunit`)

1. Which exact NUnit runner versions and executable paths must be supported?
2. Please provide the Bamboo task export and a representative test project.
3. Is Windows C# Test Intelligence a hard POC requirement?
4. What does the inventory label “Visual Studio 2025” refer to?

Harness will derive assemblies, adapters, categories, filters, arguments, result paths, conversion, and failure handling from the supplied configuration and project.

## MSTest and VSTest (`plugin.dotnet:mstest`)

1. Which exact MSTest/VSTest runner versions and executable paths must be supported?
2. Please provide the Bamboo task export, a representative test project, and any `.runsettings` files.
3. Is Windows C# Test Intelligence a hard POC requirement?
4. What does the inventory label “Visual Studio 2025” refer to?

Harness will determine whether to use `dotnet test`, `vstest.console.exe`, or another supported mode and will derive adapters, filters, collectors, reports, and conversion behavior from the evidence.

## Artifact Download (`bamboo-artifact-downloader-plugin:artifactdownloadertask`)

1. Please provide the producer and consumer Bamboo plan/job exports, including artifact definitions and download tasks.
2. Which artifact repository should be the system of record after migration?
3. Are current “latest successful” artifact lookups mandatory, or can Harness pass an explicit version and digest?
4. What retention policy must be preserved?

Harness will derive producer selectors, branches, artifact names, destinations, and failure behavior from the exports.

## Repository Commit, Push, Tag, and Branch Operations (`plugins.vcs:task.vcs.commit`, `plugins.vcs:task.vcs.push`, `plugins.vcs:task.vcs.tagging`, `plugins.vcs:task.vcs.branching`)

1. Are signed commits/tags, Git LFS, arbitrary remotes, or force-push hard POC requirements?

POC prerequisite: the credential already configured on the selected codebase connector must have permission to perform the required Git writes. Harness can reuse and persist that credential during the build, but it cannot add repository permissions to it.

Harness will derive the commands, refs, messages, branch/tag behavior, and rerun handling from the Bamboo exports. Native Clone Codebase can persist credentials for later steps, and Harness will reuse its existing Windows `harness/drone-git` image, so the customer does not need to select or supply another Git for Windows image.

## Xcode Unlock Keychain (`bamboo-xcode-plugin:unlockkeychain`)

1. Is Xcode/keychain signing a POC blocker even though it requires a separate macOS execution lane?
2. Which exact Xcode and macOS versions must be supported?
3. Please provide the Bamboo Xcode/keychain task export and a sanitized description of the current signing flow.
4. Does the current flow unlock an existing keychain or create/import an ephemeral certificate and provisioning profile?

Harness will derive the keychain commands, search-path setup, timeout, concurrency, and cleanup behavior without requesting private keys or production credentials.

## Build Warnings Parser (`atlassian-bamboo-warnings:task.warnings.parser`)

1. Which Bamboo warnings plugin/parser version is used?
2. Please provide the Bamboo task export and one representative input log or report.
3. Is a Harness summary plus downloadable report sufficient, or are clickable file/line findings, baselines, or historical trends hard POC requirements?

Harness will derive parser type, file globs, encoding, severity mapping, thresholds, and failure behavior from the task export and sample.

## Artifactory Maven and Gradle (`bamboo-artifactory-plugin:maven3Task`, `bamboo-artifactory-plugin:artifactoryGradleTask`)

1. Which exact JFrog Artifactory/plugin, JFrog CLI, JDK, Maven, and Gradle versions must be supported?
2. Please provide the Bamboo task exports and representative Maven/Gradle project files, including wrappers where present.
3. Is a non-production JFrog tenant available for testing?
4. Which authentication method is approved for the POC?

Harness will derive repository configuration, goals/tasks, build-info fields, properties, outputs, proxy, and private CA behavior from the exports and environment information.

## Artifactory Generic Resolve/Download (`bamboo-artifactory-plugin:artifactoryGenericResolveTask`)

1. Which exact JFrog Artifactory/plugin and JFrog CLI versions must be supported?
2. Please provide the Bamboo task export and all referenced file specs.
3. Is a non-production JFrog tenant available for testing?
4. Which authentication method is approved for the POC?

Harness will derive patterns, selectors, destination layout, retries, parallelism, no-match behavior, checksums, and outputs from the export and file specs.

## Artifactory npm and Build Info (`bamboo-artifactory-plugin:artifactoryNpmTask`, `bamboo-artifactory-plugin:artifactoryPublishBuildInfoTask`)

1. Which exact JFrog Artifactory/plugin, JFrog CLI, Node, and npm versions must be supported?
2. Please identify the Node version paired with each required npm version.
3. Please provide the Bamboo task exports and representative `package.json` and lockfiles.
4. Can EOL npm 5/6 combinations be upgraded, or are they hard POC blockers?
5. Is a non-production JFrog tenant available for testing?

Harness will derive npm commands, registries, scopes, build-info behavior, native-build requirements, outputs, proxy, and private CA behavior from the evidence.

## Maven POM Value Extractor (`maven-pom-parser-plugin`)

1. Which exact Bamboo POM extractor/plugin version is used?
2. Please provide the Bamboo task export and representative POM files.
3. Must values be resolved from the effective POM, or is the raw POM sufficient?

Harness will derive POM paths, GAV/custom expressions, output names, prefixes, `-SNAPSHOT` handling, and cross-stage output requirements from the exports.

## Node.js, npm, gulp, grunt, and bower (`bamboo-nodejs-plugin:*`)

1. Which exact Node and npm versions must be supported?
2. Which versions can be upgraded?
3. Please provide the Bamboo task exports and representative `package.json` and lockfiles.
4. Are global gulp, grunt, or bower installations hard requirements, or are these project-local dependencies?

Harness will derive commands, package scripts, working directories, registry settings, native-module prerequisites, cache paths, reports, and outputs from the supplied evidence.

## ScriptRunner Groovy (`groovyrunner:scriptrunner.generic`)

1. Which exact Groovy and JDK versions must be supported?
2. Please provide the Bamboo task exports and sanitized copies of the scripts selected for the POC.
3. Which scripts are hard POC blockers?

Harness will inspect the scripts to identify Bamboo API dependencies, inputs, outputs, and side effects and will determine which scripts run directly versus require a Harness-specific rewrite.

## Cucumber Reports (`cucumber-bamboo-plugin`)

1. Which exact Bamboo Cucumber plugin and Cucumber report-format versions are used?
2. Please provide the Bamboo task export and representative result files.
3. Can the test runner emit JUnit XML, or is legacy Cucumber JSON mandatory?
4. Are Jira actions or a dedicated Cucumber report/HTML view hard POC requirements?

Harness will derive globs, tags, thresholds, no-result behavior, counts, and report handling from the export and samples.

## qTest Publisher (`qtest-plugin-for-bamboo`)

1. Which exact qTest product/version and Bamboo qTest plugin version are used?
2. Please provide the Bamboo task export and representative sanitized JUnit result files.
3. Which authentication method is approved for the POC?
4. Is a non-production qTest tenant available for end-to-end testing?
5. Should missing qTest objects be created automatically, and should publication failure fail the pipeline?

Harness will derive project, release, build, cycle/suite, environment, grouping, glob, attachment, and output mappings from the export and test tenant.

## Evidence handling

Do not provide production credentials, private signing keys, or unnecessary proprietary source. Minimal sanitized examples are sufficient. Harness will request follow-up information only when the supplied export, capability inventory, project files, and samples do not determine a required implementation detail.
