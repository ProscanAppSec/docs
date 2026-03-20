# CI/CD Pipelines

Proscan integrates with your CI/CD pipeline to scan code on every commit, pull request, or scheduled build. If a scan finds issues that violate your quality gate, it can fail the build.

## Supported Platforms

- GitHub Actions
- GitLab CI
- Jenkins
- Azure Pipelines
- CircleCI
- Travis CI
- Bamboo
- TeamCity
- Drone CI
- AWS CodePipeline

## How It Works

1. Add a scan step to your pipeline configuration
2. The step calls the Proscan API to trigger a scan
3. Proscan runs the scan and returns results
4. If findings exceed your quality gate thresholds, the step exits with a non-zero code and the build fails
5. Results are available in the Proscan dashboard and optionally as a SARIF upload to your source control platform

## Quality Gates

Quality gates define what constitutes a passing scan. Common configurations:

- Fail on any critical or high severity finding
- Fail if new findings are introduced (compared to the baseline)
- Fail if secrets are detected
- Fail if SCA finds dependencies with known critical CVEs

Gates are configured in the Proscan web interface and enforced automatically during CI/CD scans.

## SARIF Integration

Proscan exports results in SARIF 2.1.0 format, which integrates directly with:

- GitHub Code Scanning (appears in the Security tab)
- GitLab Security Dashboard
- Azure DevOps code analysis

This puts findings directly in your pull request review workflow.

## GitHub Action

A pre-built GitHub Action is available at [Proscan-hub/action](https://github.com/Proscan-hub/action). See the [action documentation](https://github.com/Proscan-hub/action) for setup instructions.

## Configuration Examples

See the [integrations repository](https://github.com/Proscan-hub/integrations) for pipeline configuration examples for each supported platform.
