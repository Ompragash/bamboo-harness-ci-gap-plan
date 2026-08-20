# Ant

## Customer need

The customer runs Ant builds on Windows. Harness must provide a structured task for selecting a supported Java profile, build file, targets, properties, `ANT_OPTS`, working directory, environment, and JUnit report paths.

Ant must reuse the same Harness-owned Java runtime family as Maven. Harness should not create a separate JDK installation system or a Java x Ant image matrix.

## How Bamboo handles it

Bamboo administrators install JDK and Ant versions on long-lived Windows agents and register them as capabilities. The Ant task requires the selected capabilities, so Bamboo assigns the job to an agent that already contains both tools.

The task adds a structured build file, targets, arguments, environment, working directory, and JUnit result configuration around the installed Ant executable.

```text
Bamboo selects Windows agent with JDK + Ant already installed
-> Ant task receives build.xml, targets, properties, and options
-> installed Ant runs
-> exit status and JUnit reports return to Bamboo
```

## Harness implementation

Recommendation: build a thin Harness Ant plugin on the Harness Windows Java runtime family.

The Harness step abstraction selects the internal Java 8, 11, 17, or 21 runtime profile. Each supported profile contains one Harness-supported Ant installation and the plugin launcher. The user chooses the Java profile and Ant task inputs, not an image tag.

```text
Harness Ant Plugin
-> Harness Windows Java 17 runtime with supported Ant
-> build.xml + clean test package
-> Harness logs and JUnit results
```

Proposed plugin inputs: Java profile, Ant version where more than one is supported, build file, targets, properties, arguments, `ANT_OPTS`, working directory, environment, and report paths.

The existing `drone-ant` PR is not ready to use. It is unmerged, exposes only goals, installs unpinned JDK 8 and Ant packages, and has a failing empty-goals test. Harness can reuse any useful command-building code, but the supported plugin needs the full Ant task contract, pinned Harness-owned images, tests, signing, and release ownership.

## What we still need to confirm

- Which Java/Ant versions are POC requirements?
- Which build files, targets, properties, `ANT_OPTS`, and report paths are active?
- Are custom Ant distributions or third-party Ant tasks required?

## Customer position

- Harness will provide Ant as a structured Windows Kubernetes task.
- Ant will reuse Harness-maintained Java runtime profiles.
- Common JDK and Ant binaries will already be present when the container starts.
- Legacy Ant versions require an explicit Harness support decision.

## Sources

- [Atlassian Ant task](https://confluence.atlassian.com/display/BAMBOO/Ant)
- [`drone-ant` PR 1](https://github.com/harness-community/drone-ant/pull/1)
- [Eclipse Temurin supported platforms](https://adoptium.net/supported-platforms)
