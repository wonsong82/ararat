# ADR-014: GitHub Actions CI/CD

**Status**: Accepted  
**Date**: 2026-02-25  
**Deciders**: Project owner, AI agent

## Context

Ararat needs a CI/CD pipeline for automated testing, linting, type-checking, Docker image builds, and deployment to AWS ECS. The pipeline must integrate with the source control platform and support the monorepo structure.

## Decision

Use **GitHub Actions** for CI/CD, with source control on **GitHub**.

## Alternatives Considered

- **GitLab CI**: More powerful pipeline features (DAG, parent-child pipelines) and built-in container registry. However, requires GitLab platform, and the team already uses GitHub. GitLab CI's advanced features are not needed for Ararat's pipeline.

## Consequences

**Positive:**
- Native integration with GitHub (no external CI service needed)
- Generous free tier for open and private repositories
- Large marketplace of pre-built actions (AWS deploy, Docker build, Node.js setup)
- YAML-based workflow files live in the repository alongside code
- Matrix builds for testing across Node.js versions

**Negative:**
- Less powerful than GitLab CI for complex pipeline DAGs (not needed for this project)
- Limited built-in container registry (use AWS ECR instead)

**Affects**: [21-infrastructure](../21-infrastructure.md)