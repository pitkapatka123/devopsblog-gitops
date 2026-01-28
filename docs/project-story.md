
---

## Project Objective

This is a "graduation project" for the Telerik DevOps Upskill programme, the culmination of the past 3 and a half months. Thanks to the presentations, lessons and Telerik lecturers I was able to level up and grow into a stable DevOps mindset. The objective of this project was to apply what I alrady knew, going into the course, combine that with what I learned form Telerik and build a production-style DevOps platform for a containerized web application.  

The focus was on applying real-world DevOps practices including CI/CD automation, GitOps deployment, cloud security, observability, and operational decision-making.

The project intentionally emphasizes:
- Practical implementation over theory
- Real failure scenarios and recovery
- Security by design
- Clear architectural tradeoffs
- End-to-end ownership of the delivery pipeline

---

## Final Architecture Overview

- Flask-based web application
- Deployed on AWS using Kubernetes (EKS)
- Separate **dev** and **prod** environments
- Public access via **AWS Application Load Balancer**
- TLS termination using **AWS Certificate Manager**
- DNS managed with **Route53**
- Container images built with Docker
- Images stored in **Amazon ECR**
- Deployments managed using **GitOps (Argo CD)**
- Immutable deployments using image digests
- Application content stored in **Amazon S3**
- No secrets stored in repositories or images

---

## Repository Map

### Application Repository
Responsible for:
- Flask application source code
- Blog content (`content/`)
- Dockerfile
- CI workflows
- Snyk and SonarCloud scans (SAST)
- Image build and push to ECR
- Triggering GitOps deployments

### GitOps Repository
Responsible for:
- Kubernetes manifests (dev and prod)
- Argo CD application definitions
- Deployment configuration
- Content promotion workflows
- ZAP security scanning (DAST)
- GitOps-based promotion logic

---

## Definition of Done

### Planning & Design
- Architecture defined and documented
- Tooling stack selected and justified
- GitOps model chosen
- Responsibilities between Terraform and Kubernetes defined
- Separation of concerns established (app, infra, delivery)

---

### Infrastructure & Kubernetes
- EKS clusters provisioned for dev and prod
- Node groups created and verified
- Networking configured correctly
- Core addons deployed and validated
- Argo CD installed and reachable
- GitOps repository initialized and connected

---

### GitOps & Deployment Model
- GitOps adopted as the single deployment mechanism
- Argo CD configured to reconcile desired state
- Application deployed via GitOps only
- Manual kubectl usage strictly avoided for deployments
- Drift detection and reconciliation verified

---

### CI/CD Implementation
- GitHub Actions configured for CI
- OIDC authentication to AWS implemented
- Docker images built in CI
- Images pushed to Amazon ECR
- Digest-based image deployments enforced
- GitOps repository updated automatically via PR

---

### Application & Content Architecture
- Application containerized
- Blog content separated from image
- Content stored in Amazon S3
- Application reads content dynamically
- Separate dev and prod content paths
- Content promotion flow implemented

---

### Security & Identity
- OIDC used for GitHub → AWS authentication
- No long-lived AWS credentials used
- IRSA implemented for pod-level AWS access
- Separate IAM roles for dev and prod
- HTTPS enforced via ACM
- Secrets stored outside repositories
- Public endpoints documented as intentional

---

### Security Testing
- Snyk integrated for dependency scanning
- SonarCloud used for static analysis
- OWASP ZAP baseline scan enabled
- ZAP enforced as pipeline gate
- Explicit allowlist maintained via `zap-rules.conf`

---

### Networking & TLS
- Domain registered and configured
- Route53 DNS records created
- ACM certificates issued
- TLS termination handled by ALB
- HTTP to HTTPS redirection enforced
- ALB configuration validated

---

### Observability
- CloudWatch logging enabled
- ALB metrics monitored
- Basic health endpoints implemented
- No over-engineered monitoring stack
- Observability aligned with system complexity

---

### Stability & Incident Handling
- Argo CD outage identified and recovered
- IRSA trust policy issue resolved
- EKS addon ordering issue resolved
- Recovery procedures documented
- Root causes analyzed and recorded

---

### Documentation & Finalization
- Architecture documented
- CI/CD flow documented
- Security model documented
- Tradeoffs explicitly stated
- Lessons learned captured
- Project prepared for portfolio use

---

## Major Incidents & Root Cause Analysis

### Argo CD Outage
- **Cause:** Attempted to manage Argo configuration via GitOps
- **Impact:** Argo became unavailable
- **Resolution:** Manual recovery and removal from GitOps control
- **Lesson:** Not all components should be managed declaratively

### IRSA Trust Policy Failure
- **Cause:** Incorrect namespace in IAM trust relationship
- **Impact:** Application failed to access S3
- **Resolution:** Corrected trust policy
- **Lesson:** IRSA is strict and namespace-sensitive

### EKS Addon Ordering Issue
- **Cause:** Addons applied before node groups existed
- **Impact:** CoreDNS and networking failures
- **Resolution:** Corrected Terraform ordering
- **Lesson:** EKS addons depend on node availability

### ZAP Enforcement Issues
- **Cause:** Security rules triggered pipeline failures
- **Resolution:** Explicit allowlist via `zap-rules.conf`
- **Lesson:** Security tooling must be tuned, not disabled


---

## CI/CD + GitOps Flow

### Image Pipeline
1. PR created
2. SonarCloud & Snyk scans
3. Image built
4. Image pushed to ECR
5. Digest generated
6. GitOps PR created

### Deployment Pipeline
1. GitOps repo updated
2. Argo CD syncs changes
3. Application deployed
4. ZAP validation executed
5. Promotion to prod after success

### Content Pipeline
1. Content pushed to repository
2. Uploaded to S3 (dev)
3. Validation workflow runs
4. Content promoted to prod
5. App reads content dynamically

---

## Observability

- CloudWatch Logs
- ALB metrics (latency, 5xx, request count)
- Kubernetes events


---

## Future Improvements

- Terraform execution in CI (with safeguards)
- Private EKS API endpoint
- Bastion or VPN access
- Advanced monitoring (Prometheus/Grafana)
- Canary deployments

---

## Lessons Learned

- GitOps requires discipline and clear ownership
- IRSA configuration is easy to misconfigure
- EKS addons depend on node availability
- Security tooling must be tuned, not blindly enforced
- Immutable images prevent entire classes of bugs
- Observability should match system complexity
- Simplicity beats overengineering
- Debugging real failures teaches more than tutorials

---
