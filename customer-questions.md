# Customer questions for the Windows POC

These eight answers determine which Harness-owned runtime profiles and plugins must be built for the POC.

1. Which capabilities in this plan are hard POC acceptance blockers, and can you provide one exported Bamboo task configuration plus a minimal representative project/report for each blocker?

2. Which Windows Server LTSC version, container isolation mode, and CPU architecture will the Kubernetes worker nodes use?

3. Which exact JDK versions and Java distributions are required, and is JDK 7 a hard POC requirement?

4. Which Maven, Ant, and Groovy versions are required; which Maven repositories contain `mvnw.cmd`; and which Java build task inputs are actively used?

5. Which exact Node/npm pairs are required, which can be upgraded, and do any projects depend on global gulp, grunt, bower, or native-module Build Tools?

6. Which solution/project types, MSBuild versions, Visual Studio workload/component IDs, targeting packs, SDKs, architectures, and `devenv.exe` operations are hard POC requirements?

7. Which NUnit/MSTest runners, frameworks, adapters, filters/settings, report formats, and Windows Test Intelligence behavior are required?

8. Can builds access approved public sources, or must packages use internal mirrors/proxy/private CA; and are non-production JFrog and qTest tenants available for integration testing?

## Requested evidence

- Bamboo Specs or task exports for selected blockers.
- One sanitized success and expected-failure log per selected task family.
- Minimal Java, Node, .NET, NUnit/MSTest, POM, Cucumber, warning, and JUnit fixtures only where selected.
- Approved runtime licenses or vendor contracts for any non-standard Java, Node, Visual Studio, or test-runner version.
- Target Windows Kubernetes details and non-production integration access.

Secrets, production signing keys, proprietary source, and production credentials are not required.
