# Code Quality

The code quality scanner measures maintainability alongside security. It tracks complexity, duplication, and technical debt — giving your team a clear picture of code health.

## What It Measures

### Complexity

- **Cyclomatic complexity** — counts the number of independent paths through a function
- **Cognitive complexity** — measures how difficult a function is to understand (accounts for nesting, control flow, and recursion)

Both metrics are calculated per function, so you can identify specific areas that need refactoring.

### Duplication

Detects copy-pasted code across files using Type-1 (exact) and Type-2 (renamed variables) clone detection. Duplication is a maintenance risk — bugs fixed in one copy often go unfixed in others.

### Technical Debt

Uses the SQALE model to estimate remediation effort. Each issue is assigned a time cost, and the total across your codebase is expressed as a debt rating from A (minimal) to E (significant).

## Quality Gates

You can define quality thresholds that block CI/CD builds if they're not met. For example:

- No new critical findings
- Cognitive complexity under a set limit
- Duplication below a certain percentage
- Technical debt rating at C or better

Quality gates are configurable per project.

## Language Support

Code quality analysis supports 30+ languages.

## Usage

1. Create a scan and select **Code Quality**
2. Point it at your repository
3. Run the scan

Results show complexity hotspots, duplicated blocks, and the overall technical debt score. Over time, you can track whether code quality is improving or degrading across releases.
