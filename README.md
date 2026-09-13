# Centralized CI/CD Infrastructure Architecture

The centralized CI/CD infrastructure shifts pipeline maintenance from individual service repositories to a single, dedicated workflow repository. By leveraging GitHub Actions Reusable Workflows (`workflow_call`), engineering teams maintain uniform standards across all Go microservices without duplicating verbose configuration files.

## Core Use Cases & Benefits

| Dimension | Distributed CI/CD Approach | Centralized Shared Workflows |
| :--- | :--- | :--- |
| **Maintenance & Overhead** | Updates must be manually copy-pasted across dozens of individual repositories. | Single-point updates in the shared repository instantly propagate to all consuming services. |
| **Security & Compliance** | Vulnerable linter rules, insecure build args, or outdated Go versions easily slip through. | Mandatory security checks, isolated linter environments, and strict Go toolchain versions are centrally enforced. |
| **Developer Onboarding** | New services require writing complex GitHub Action YAMLs from scratch. | New repositories spin up in minutes by referencing a single-line caller workflow. |

## End-to-End Execution Flow

The shared architecture manages the software delivery lifecycle through isolated, automated stages:

* **Pull Request Validation:** When a developer opens a PR targeting `main`, the workflow provisions an isolated runner, sets up the Go environment, executes containerized `golangci-lint` with strict timeout controls, and runs concurrent race-condition test suites (`go test -v -race -cover ./...`).
* **Container Compilation & Publishing:** Upon merging code into `main`, the pipeline builds a production-ready, multi-stage Alpine Linux image running under a secure non-root user, automatically pushing the artifact to the GitHub Container Registry (`ghcr.io`) tagged with the precise Git commit SHA and `latest`.

## Blueprint Integration for Service Repositories

Adopting this architecture across microservices requires minimal local footprint. Consuming repositories replace their local pipelines with a lightweight caller configuration at `.github/workflows/ci-cd.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  call-pipeline:
    uses: MorningBlossom/shared-workflows/.github/workflows/go-ci-cd.yml@main
    with:
      go-version: '1.27'
      image-name: 'service-name'