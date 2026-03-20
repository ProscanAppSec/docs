# Binary Analysis

The binary scanner examines compiled executables and bytecode without requiring source code. It checks for known vulnerabilities, security hardening gaps, and embedded secrets.

## Supported Formats

| Format | Description |
|--------|------------|
| JAR / WAR | Java archives |
| DLL / EXE | Windows PE binaries |
| APK / DEX | Android packages |
| ELF | Linux executables |
| PE | Windows portable executables |
| Mach-O | macOS binaries |

## What It Checks

### Hardening Analysis

The scanner verifies that standard security hardening measures are in place:

- ASLR (Address Space Layout Randomization)
- DEP / NX (Data Execution Prevention)
- Stack canaries
- PIE (Position Independent Executable)
- Code signing

### Vulnerability Detection

- Lifts bytecode to an intermediate representation for analysis
- Builds call graphs and performs taint tracking across function boundaries
- Matches embedded libraries and components against CVE databases

### Binary SCA

Identifies third-party libraries compiled into the binary and checks them for known vulnerabilities. Generates an SBOM even without access to the original build files.

## When to Use It

Binary analysis is useful when you need to assess the security of software you don't have source code for — vendor-supplied binaries, third-party components, legacy applications, or mobile app packages.

## Usage

1. Create a scan and select **Binary Analysis**
2. Upload the binary file or provide a path
3. Run the scan

Results include hardening status, detected vulnerabilities with CWE mappings, and any embedded library CVEs.
