# IaC Security — Infrastructure as Code

The IaC scanner checks your infrastructure templates for misconfigurations before they're deployed. It catches issues like overly permissive IAM policies, unencrypted storage, public-facing resources that should be private, and missing logging — before any of it reaches production.

## Supported Template Types

| Template Type | File Extensions |
|--------------|----------------|
| Terraform | `.tf`, `.hcl`, `.tfvars`, `.tf.json` |
| Kubernetes | `.yaml`, `.yml` (K8s manifests) |
| Dockerfile | `Dockerfile` |
| CloudFormation | `.json`, `.yaml` (CFN templates) |
| Ansible | `.yaml`, `.yml` (playbooks) |
| Helm | Helm chart structure |

## What It Checks

The scanner evaluates templates against CIS benchmarks and security best practices:

- **AWS** — CIS AWS Foundations Benchmark
- **GCP** — CIS GCP v2.0
- **Azure** — CIS Azure Foundations
- **Kubernetes** — CIS Kubernetes Benchmark
- **Docker** — CIS Docker Benchmark

Common findings include:

- Storage buckets or blobs without encryption
- Security groups with unrestricted ingress (0.0.0.0/0)
- IAM roles with excessive permissions
- Containers running as root
- Missing network policies
- Logging and monitoring not enabled
- Secrets passed through environment variables

## Custom Policies

You can define your own rules using pattern-based definitions. Each rule specifies what to look for, a severity level, and a remediation message. This lets you enforce organization-specific standards alongside the built-in checks.

## Terraform Plan Analysis

For Terraform users, the scanner can analyze plan output to detect configuration drift and catch issues that only appear during the plan phase — not just in static template files.

## Usage

1. Create a scan and select **IaC**
2. Point it at a repository containing infrastructure templates
3. Run the scan

Results show the misconfiguration, the file and resource where it was found, the relevant CIS benchmark control, and a suggested fix.
