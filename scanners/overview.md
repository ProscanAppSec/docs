# Scanner Overview

Proscan includes twelve scanner modules. Each one targets a different layer of your application stack. They can run independently or together — findings are correlated across scanners to reduce noise and surface real risk.

| Scanner | What It Analyzes |
|---------|-----------------|
| [SAST](sast.md) | Source code — static analysis across 60+ languages |
| [DAST](dast.md) | Running web applications — dynamic black-box testing |
| [ProScan Deep](proscan-deep.md) | Advanced dynamic scanning with built-in pen test tools |
| [SCA](sca.md) | Open-source dependencies and license risk |
| [Secrets](secrets.md) | Hardcoded credentials, API keys, and tokens |
| [IaC](iac.md) | Infrastructure-as-code templates and configs |
| [Container](container.md) | Container images and Dockerfile analysis |
| [API](api.md) | REST and GraphQL endpoint testing |
| [AI/LLM](ai-llm.md) | AI model and LLM endpoint security |
| [Binary](binary.md) | Compiled binaries and bytecode |
| [Network](network.md) | Host discovery, port scanning, service fingerprinting |
| [Code Quality](code-quality.md) | Complexity, duplication, and maintainability |

## Cross-Scanner Correlation

When multiple scanners run against the same target, Proscan correlates their findings. For example, if SAST identifies a SQL injection in code and DAST confirms it's exploitable at runtime, the finding is grouped and marked with higher confidence. If SCA flags a vulnerable dependency and SAST confirms the vulnerable function is actually called, that's surfaced as a confirmed risk rather than a theoretical one.

## False Positive Elimination

All findings pass through a multi-layer verification pipeline before reaching your dashboard. This includes pattern-based filters, contextual analysis, confidence scoring, and cross-scanner validation. The goal is to show your team only what's real, so they can focus on fixing rather than triaging noise.

## Scan Scheduling

Scans can be triggered manually, on a schedule, or through CI/CD pipeline integration. See [CI/CD Pipelines](../integrations/cicd.md) for details on automated scanning.
