# Customer questions for the Windows POC

These eight answers determine the explicit Harness image tags, templates, and Plugin images required for the POC.

1. Which capabilities are hard POC acceptance blockers, and can you provide one exported Bamboo task plus a minimal representative project/report for each blocker?

2. Which Windows Server LTSC version, container isolation mode, and CPU architecture will the Kubernetes worker nodes use?

3. Which exact JDK, Maven, Ant, and Groovy combinations are active, and is JDK 7 a hard POC requirement?

4. Is reproducing Bamboo's structured Maven/Ant form a hard acceptance requirement, or is an approved native Run Step Template with explicit image and command inputs acceptable?

5. Which exact Node/npm pairs are required, which can be upgraded, and do any projects require global gulp/grunt/bower or native-module Build Tools?

6. Which solution/project types, MSBuild versions, Visual Studio workload/component IDs, targeting packs, SDKs, architectures, and `devenv.exe` operations are hard POC requirements?

7. Which NUnit/MSTest runners, frameworks, adapters, filters/settings, report formats, and Windows Test Intelligence behavior are required?

8. Which JFrog, Cucumber, warnings, POM, and qTest behaviors are POC blockers; are test tenants available; and must runtime/package downloads use internal mirrors, proxy, or private CA?

## Requested evidence

- Bamboo Specs or task exports for selected blockers.
- One sanitized success and expected-failure log per selected task family.
- Minimal Java, Node, .NET, NUnit/MSTest, POM, Cucumber, warning, and JUnit fixtures only where selected.
- Approved runtime licenses or vendor contracts for any non-standard Java, Node, Visual Studio, or test-runner version.
- Target Windows Kubernetes details and non-production integration access.

Secrets, production signing keys, proprietary source, and production credentials are not required.
