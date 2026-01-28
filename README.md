# DevOpsBlog — GitOps Repository

## Overview
This repository contains the GitOps configuration for DevOpsBlog and acts as the single source of truth for Kubernetes deployments.

This repo includes:
- Kubernetes manifests for dev and prod
- Argo CD Application definitions
- Argo CD Apps-of-Apps for dev and prod environments
- GitHub Actions workflows for validation and promotion
- ZAP configuration (`zap-rules.conf`)

This repo does not include:
- Application source code (owned by the application repository)
- Terraform/infrastructure code

## GitOps model
- Argo CD continuously reconciles Kubernetes state from this repository.
- All deployment changes happen via Git commits.
- Rollbacks are performed by reverting Git commits.
- Drift is corrected automatically by Argo CD.

## Image eployment workflow
1. The application repository builds and pushes an image to ECR and captures its digest.
2. The application repository opens a pull request to this repository updating the dev deployment to the new digest.
3. Pull-request is auto-merged thanks to autobot-merge-gitops.yaml (after socket security check)
4. Argo CD syncs and updates the running dev environment.
5. After merge, dev-validation.yaml runs against dev, scans via ZAP, then promotion updates prod manifests.
5. Argo CD syncs prod and validation runs against prod via prod-validation.yaml, again with ZAP baseline scan

## Content workflow (S3)
1. A content change in the application repository uploads content to S3 under the dev prefix.
2. A validation workflow is dispatched to this repo (GitOps) to scan the dev environment again via content-validation.yaml (smoke + ZAP baseline).
3. If validation succeeds, a the content-copy.yaml workflow promotes content from dev to prod in S3.
4. A prod validation workflow runs after promotion via prod-validation.yaml

## ZAP enforcement
- OWASP ZAP Baseline is enforced in CI.
- Pipelines fail on findings.
- Specific accepted rules are allowlisted via `zap-rules.conf`.
- Informational findings are intentionally ignored; only actionable findings are gated.

## Security notes
- No secrets are stored in this repository.
- Cross-repo authentication uses a GitHub App token stored in AWS Secrets Manager, resulting in ***Full*** automation.
- AWS access is via OIDC for workflows and IRSA for in-cluster workloads.
- TLS is handled at the ALB using ACM certificates and HTTP to HTTPS redirect is enforced.

## Design decisions
- No Helm or Kustomize due to time constraints. Helm implementation is planned in the future

## How to use this repo
- Deploy Argo CD to a Kubernetes cluster.
- Point an Argo CD Application at this repo and the appropriate environment path (dev/prod).
- Ensure your cluster has:
  - AWS Load Balancer Controller installed
  - IRSA configured for required service accounts
  - Access to ECR for pulling images
  - Correct S3 bucket/prefix configuration for content
- Merge changes via pull requests to trigger GitOps reconciliation.

## Architecture/Workflow Overview


![Main Pipeline Diagram](docs/flow.png)

![Content Pipeline Diagram](docs/contentflow.png)
