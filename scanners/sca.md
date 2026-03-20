# SCA — Software Composition Analysis

SCA identifies known vulnerabilities in your open-source dependencies. It maps your full dependency tree, matches components against vulnerability databases, and assesses whether vulnerable code is actually reachable in your application.

## How It Works

1. Proscan parses your project's dependency files (package.json, requirements.txt, go.mod, pom.xml, etc.)
2. It resolves the full transitive dependency tree — not just direct dependencies
3. Each component is matched against known CVE records
4. Reachability analysis determines if the vulnerable code path is actually used

## Supported Ecosystems

SCA covers 17+ package ecosystems, including:

npm, PyPI, Maven, Go modules, Cargo (Rust), NuGet, Composer (PHP), CocoaPods, RubyGems, Gradle, pip, Poetry, Yarn, PNPM, Hex (Elixir), pub (Dart), and more.

## Reachability Analysis

Not every vulnerable dependency is actually dangerous. A library might have a known vulnerability in a function your code never calls. SCA performs reachability analysis at three levels:

- **Function-level** — the vulnerable function is directly called in your code
- **Import-level** — the module containing the vulnerability is imported but the specific function isn't called
- **Not imported** — the vulnerable package is in your dependency tree but never loaded

This distinction helps your team prioritize real risk over theoretical exposure.

## SBOM Generation

SCA generates Software Bills of Materials in standard formats:

- **CycloneDX 1.5**
- **SPDX 2.3**
- **VEX** (Vulnerability Exploitability eXchange)

These are useful for compliance, supply chain transparency, and sharing dependency information with customers or auditors.

## License Detection

Beyond vulnerabilities, SCA also identifies the licenses of your dependencies. A policy engine lets you define which licenses are acceptable, which need review, and which should be flagged. This is important for organizations with legal requirements around open-source usage.

## Usage

1. Create a scan and select **SCA**
2. Point it at your repository
3. Run the scan

Results show each vulnerable dependency, the CVE details, affected versions, a fixed version (if available), and the reachability assessment.
