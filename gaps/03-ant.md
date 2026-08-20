# Ant

## Customer need

The customer runs Ant builds on Windows. Harness must support the required JDK and Ant versions, build file, targets, properties, `ANT_OPTS`, working directory, environment, and JUnit report paths in Kubernetes Windows containers.

## How Bamboo handles it

Bamboo administrators install JDK and Ant on long-lived agents and register both as capabilities. The Ant task selects those capability labels, which become job requirements and restrict execution to matching agents.

The task adds structured fields around the installed executable: build file, targets, arguments, environment, working directory, and JUnit results.

```text
Bamboo task requires JDK + Ant capabilities
-> Bamboo selects a matching Windows agent
-> task constructs the Ant command
-> installed Ant runs and Bamboo collects results
```

## Harness implementation

Recommendation: use the Harness-maintained Windows Java build image through a native Run step. Do not build an Ant plugin for the POC.

The pipeline explicitly references the tag containing the required JDK, for example `harness/windows-java-build:temurin17`. The same image family used for Maven contains one supported Ant distribution and Groovy. Keeping these relatively small Java build tools together avoids separate Maven, Ant, and Groovy image families while retaining one JDK per tag.

```text
Harness Run step
-> explicit harness/windows-java-build:temurin17 image
-> ant -f build.xml clean test package
-> Harness logs, status, outputs, and JUnit report paths
```

A reusable Run Step Template standardizes the image tag, command pattern, working directory, environment, secrets, and report paths. Ant properties and targets remain normal command inputs. This reproduces the useful Bamboo behavior without maintaining plugin code that only assembles an Ant CLI.

The open `drone-ant` PR does not change this recommendation. It is unmerged, exposes only goals, installs unpinned JDK 8 and Ant packages, and is not needed when the prepared image and Run template provide parity.

An additional image tag is created only when an active project requires an incompatible Ant/JDK pair. Harness does not install Ant or Java during each pipeline execution.

## What we still need to confirm

- Which JDK and Ant combinations are active POC requirements?
- Which build files, targets, properties, and `ANT_OPTS` are used?
- Are custom Ant distributions or third-party Ant tasks required?

## Customer position

- Ant uses a native Run step with an explicit Harness-owned Java build image.
- Maven, Ant, and Groovy share the same bounded image family.
- A dedicated Ant plugin is not required for the POC.
- Legacy tool combinations require an explicit support decision.

## Sources

- [Bamboo Ant task](https://confluence.atlassian.com/display/BAMBOO/Ant)
- [Bamboo executable capabilities](https://confluence.atlassian.com/display/BAMBOO/Defining%2Ba%2Bnew%2Bexecutable%2Bcapability)
- [Harness Run step](https://developer.harness.io/docs/continuous-integration/use-ci/run-step-settings/)
- [`drone-ant` PR 1](https://github.com/harness-community/drone-ant/pull/1)
