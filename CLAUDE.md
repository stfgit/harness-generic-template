# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Harness CI/CD template repository containing comprehensive pipeline configurations, stage templates, and deployment environments for building, testing, scanning, and deploying applications using Harness.io platform. The repository supports both npm/Node.js and Maven/Java workflows with progressive security hardening.

## Repository Structure

```
harness-generic-template/
├── README.md                               # Basic repository description
├── DEVSECOPS_GUIDE.md                     # Comprehensive DevSecOps usage guide
├── CLAUDE.md                              # This technical guide
├── akostash/                              # Additional template variants
├── docs/                                  # Documentation directory
│   ├── examples/USAGE_EXAMPLES.md        # Practical examples
│   └── security/                         # Security documentation
└── .harness/                             # Harness configuration directory
   ├── pipelines/                         # Pipeline definitions
   │   ├── check_stage_tplts.yaml        # Maven-based complete pipeline
   │   ├── check_stage_tplts_npm.yaml    # npm-based complete pipeline
   │   └── security_demo_pipeline.yaml    # Security demonstration pipeline
   ├── templates/                         # Reusable stage templates
   │   ├── npm_build/v0.1.yaml           # Node.js build and Docker push
   │   ├── maven_build_stage_tplt/v0.1.yaml # Java build with coverage
   │   ├── npm_pipeline/v0.1.yaml        # Complete npm pipeline template
   │   ├── mvn_pipeline/v0.1.yaml        # Complete Maven pipeline template
   │   ├── source_code_scan/             # Security source code scanning
   │   │   ├── v0.1.yaml (standard)
   │   │   ├── v0.2.yaml (enhanced)
   │   │   └── v0.3.yaml (maximum)
   │   ├── images_scan/                   # Container vulnerability scanning
   │   │   ├── v0.1.yaml (standard)
   │   │   ├── v0.2.yaml (enhanced)
   │   │   └── v0.3.yaml (maximum)
   │   ├── deploy_k8s/v0.1.yaml          # Kubernetes deployment
   │   ├── manual_approval/v0.1.yaml     # Manual approval workflow
   │   ├── jira_approval/v0.1.yaml       # JIRA-based approval
   │   ├── security_hardened_pipeline_template/v0.1.yaml # Enterprise security
   │   └── security_policies/v0.1.yaml   # Governance template
   └── envs/                              # Environment definitions
       ├── pre_production/dev/           # Development environment
       │   └── infras/infra_dev.yaml
       └── production/prod/              # Production environment
           └── infras/infra_prod.yaml
```

## Pipeline Architecture

The repository implements a comprehensive CI/CD architecture with three distinct security levels:

### Complete Pipeline Patterns

#### 1. Maven/Java Pipeline (`check_stage_tplts.yaml`)
End-to-end pipeline for Java applications targeting `stfgit/mower-java` repository:
- **Source Code Scanning**: Parallel OWASP, AquaTrivy, Gitleaks security scans
- **Build & Test**: Maven build with JaCoCo coverage (70% minimum, 80% target)
- **Container Scanning**: Docker image vulnerability assessment
- **Dev Deployment**: Kubernetes rollout to development environment
- **Manual Approval**: DevOps team approval for production
- **Production Deployment**: Automated production rollout with service reuse

#### 2. npm/Node.js Pipeline (`check_stage_tplts_npm.yaml`)
Complete pipeline for Node.js applications targeting `stfgit/nextly` repository:
- **Source Code Scanning**: Same security tools with lenient thresholds for development
- **Build & Test**: npm workflow with lint, build, test, and audit
- **Container Scanning**: Docker image security validation
- **Multi-Environment Deployment**: Dev and production deployment stages
- **Approval Workflow**: Configurable approval process

#### 3. Enterprise Security Pipeline (`security_hardened_pipeline_template/v0.1.yaml`)
Production-ready pipeline with enterprise-grade security controls:
- **Pre-Security Validation**: Pipeline initialization with strict security policies
- **Hardened Scanning**: Enhanced source code and container security scans (v0.2)
- **Zero-Tolerance Policies**: Critical vulnerabilities block deployment (0 tolerance)
- **Security Team Approval**: Two-person approval requirement
- **Compliance Framework Integration**: SOX, GDPR, CIS, NIST validation
- **Production Security Gates**: Final validation with comprehensive audit trails

## Key Template Architecture

### Build Templates

#### npm Build Template (`npm_build/v0.1.yaml`)
Sequential build process with French logging:
```bash
# Executed steps in order:
npm cache clean --force
npm install --prefer-offline --no-audit --no-fund
npx next --version                    # Version verification
npm run lint                          # ESLint validation
npm run build                         # Next.js build
npm run test:ci || echo "Tests a implementer"  # Unit tests (optional)
npm audit --audit-level=high || true # Security audit (non-blocking)
```
- **Shared paths**: `/harness/node_modules`, `/harness/.next`, `/harness/.npm`
- **Resource optimization**: 4Gi memory, 1500m CPU for Docker builds
- **Caching**: Enabled for Docker layer optimization

#### Maven Build Template (`maven_build_stage_tplt/v0.1.yaml`)
Comprehensive Java build with coverage analysis:
```bash
# Build phase
mvn clean install -DskipTests=true -Dmaven.javadoc.skip=true \
  -Dmaven.repo.local=/harness/.m2/repository -B -V

# Test with coverage
mvn test jacoco:report -B -Dmaven.repo.local=/harness/.m2/repository

# Coverage quality gate (Alpine + bc for calculations)
# Minimum: 70%, Target: 80%
```
- **Maven cache**: `/harness/.m2/repository`
- **Coverage parsing**: Advanced JaCoCo CSV analysis with per-class breakdown
- **Quality gates**: Automated coverage validation with detailed reporting
- **Docker images**: `maven:3.9.6-eclipse-temurin-21`, `alpine:latest`

### Security Templates

#### Progressive Security Scanning
**Version Strategy**: v0.1 (standard) → v0.2 (enhanced) → v0.3 (maximum)

**Source Code Scanning** (`source_code_scan/v0.1-v0.3.yaml`):
- **OWASP Dependency Check**: Configurable severity thresholds
- **AquaTrivy**: Security and compliance scanning with progressive resource allocation
- **Gitleaks**: Git secret detection with zero-tolerance policies in enterprise mode
- **Resource scaling**: Memory from 2Gi (v0.1) to 4Gi (v0.2+), CPU up to 2000m

**Container Image Scanning** (`images_scan/v0.1-v0.3.yaml`):
- **AquaTrivy Container**: Docker vulnerability scanning
- **Progressive policies**: Failure thresholds tighten from none → high → medium severity
- **Enterprise features**: v0.2+ includes enhanced scanning and stricter policies

#### Enterprise Security Controls (`security_hardened_pipeline_template/v0.1.yaml`)
**Hardened Security Configuration**:
- **Namespace isolation**: `harness-security-scan` for security operations
- **Resource allocation**: Enhanced limits (4Gi memory, 2000m CPU)
- **Vulnerability thresholds**: 
  - Critical: 0 tolerance
  - High: Maximum 5 allowed
  - Moderate audit required with JSON reporting
- **Security team approval**: Minimum 2 approvers from Security and DevSecOps teams

### Deployment Templates

#### Kubernetes Deploy (`deploy_k8s/v0.1.yaml`)
- **Deployment strategy**: Rolling deployment
- **Environment support**: dev (Docker Desktop), staging, prod
- **Infrastructure connectors**: 
  - `org.kubdesktop` for development
  - `account.k8sdocacloud` for builds
- **Service integration**: Reusable service configurations with artifact management

## Environment Configuration

### Development Environment
```yaml
# .harness/envs/pre_production/dev/dev.yaml
environment:
  type: PreProduction
  namespace: dev
  connector: org.kubdesktop
```

### Production Environment  
```yaml
# .harness/envs/production/prod.yaml
environment:
  type: Production
  namespace: production
  infrastructure: Dedicated production cluster
```

## Common Development Commands

### npm/Node.js Projects
```bash
# Local development workflow matching pipeline
npm cache clean --force
npm install --prefer-offline --no-audit --no-fund
npm run lint                    # ESLint validation
npm run build                   # Next.js/React build
npm run test:ci                 # CI-optimized testing
npm audit --audit-level=high    # Security vulnerability check

# Docker operations (matching pipeline)
docker build -t myapp:latest .
docker push stfgit/nextly:latest
```

### Maven/Java Projects
```bash
# Local development matching pipeline settings
mkdir -p ~/.m2/repository       # Local cache setup

# Build with same settings as pipeline
mvn clean install -DskipTests=true -Dmaven.javadoc.skip=true \
  -Dmaven.repo.local=~/.m2/repository -B -V

# Test with coverage (pipeline matching)
mvn test jacoco:report -B -Dmaven.repo.local=~/.m2/repository

# Coverage validation (requires bc calculator)
# View coverage: target/site/jacoco/index.html
```

### Security and Testing Commands
```bash
# Manual security scans (matching pipeline tools)
# Note: These require appropriate tool installations

# OWASP dependency check
mvn org.owasp:dependency-check-maven:check

# Container scanning with Trivy
trivy image myapp:latest

# Git secrets detection
git secrets --scan --recursive

# Coverage analysis
jacoco:check  # Validates against 70% minimum threshold
```

## Pipeline Variables and Configuration

### Core Pipeline Inputs
```yaml
# Essential variables for all templates
connectorRef: <+input>          # Docker registry connector
namespace: <+input>.default(harness-ci)
image: <+input>                 # Base Docker images
repo: <+input>                  # Docker repository path
fail_on_severity: <+input>      # Security scan thresholds

# Common connector values
org.Dockerhub                   # Docker registry
account.Github                  # Git connector  
org.kubdesktop                  # Dev Kubernetes
account.k8sdocacloud           # Build cluster
```

### Enterprise Security Variables
```yaml
# Security hardened pipeline specific
security_level: ENTERPRISE_HARDENED
compliance_frameworks: "SOX,GDPR,CIS,NIST"
manual_approval_required: true
environment: production         # Determines security policies
```

### Resource Limits by Template
```yaml
# npm builds
memory: 4Gi, cpu: 1500m        # Docker build step
memory: Default, cpu: Default   # Other steps

# Maven builds  
memory: <+input>, cpu: <+input> # All steps configurable

# Security scans
memory: 2Gi-4Gi, cpu: 500m-2000m # Progressive by version
```

## Organization Structure

```yaml
# Harness organizational hierarchy
organization: stf
project: harness_generic_template
git_integration: account.Github
docker_registry: org.Dockerhub

# Key connectors
connectors:
  - account.k8sdocacloud        # Build infrastructure
  - org.kubdesktop              # Dev Kubernetes  
  - org.Dockerhub               # Container registry
  - account.Github              # Source control

# User groups for approvals
user_groups:
  - org.DevOps_Team            # Standard approvals
  - org.DevSecOps_Team         # Security approvals
  - org.Security_Team          # Enterprise approvals
  - org._organization_all_users # Notifications
```

## Template Versioning Strategy

**Progressive Security Hardening**:
- **v0.1**: Base templates with standard security practices
- **v0.2**: Enhanced security with stricter failure policies and increased resources
- **v0.3**: Maximum security hardening for enterprise environments

**Version Selection Guidelines**:
- Development: Use v0.1 for fast feedback
- Staging: Use v0.2 for security validation  
- Production: Use security_hardened_pipeline_template with v0.2+ scans

## Pipeline Execution Patterns

### Standard Development Flow
```yaml
# check_stage_tplts_npm.yaml pattern
stages:
  1. check_sources (parallel security scans)
  2. build_and_push (npm workflow + Docker)
  3. check_images (container scanning)
  4. rollout_deploy_dev (Kubernetes deployment)
  5. approval (manual gate)
  6. prod_rollout_deploy (production deployment)
```

### Enterprise Security Flow
```yaml
# security_hardened_pipeline_template pattern
stages:
  1. pre_security_validation (initialization)
  2. source_security_scan_hardened (v0.2 scans)
  3. secure_build_package (hardened build)
  4. container_security_scan_hardened (v0.2 container)
  5. security_policy_enforcement (governance)
  6. security_team_approval (2-person approval)
  7. secure_deploy_dev (dev deployment)
  8. production_security_gate (final validation)
```

## Notification and Monitoring

### Pipeline Notifications
```yaml
# Standard notifications for all pipelines
events:
  - PipelineSuccess/PipelineFailed
  - StageStart (for approval stages)
recipients: 
  - org._organization_all_users (email)

# Enterprise security notifications
security_events:
  - Pipeline/Stage failures on security stages
  - Critical vulnerability detections
recipients:
  - Security Team, DevSecOps Team (email)
  - Slack channels (webhook integration)
```

## Development Notes

- **Expression Language**: Templates extensively use `<+input>`, `<+pipeline.sequenceId>`, `<+artifacts.primary.image>`
- **Caching Strategy**: Docker layer caching enabled, shared paths for build artifacts
- **Quality Gates**: Coverage enforcement (70% min, 80% target), vulnerability thresholds
- **Multi-language Support**: npm (Node 20), Maven (Java 21 with Eclipse Temurin)
- **Security Integration**: Zero-downtime security scanning with configurable failure policies
- **Artifact Management**: Docker images tagged with `<+pipeline.executionId>` and `latest`
- **Infrastructure**: Delegate selectors for different environments (`helm-delegate`, `helm-delegate-k8s-fat`)