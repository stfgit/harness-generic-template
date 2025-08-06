# Exemples d'Usage - Templates DevSecOps

## 🎯 Scénarios d'Utilisation Pratiques

### Scénario 1: Application Node.js - Développement Rapide
**Contexte**: Équipe de développement, déploiements fréquents, sécurité standard

```yaml
# pipeline-dev-nodejs.yaml
pipeline:
  name: "NodeJS App - Development"
  identifier: nodejs_dev_pipeline
  projectIdentifier: harness_generic_template
  orgIdentifier: stf
  
  variables:
    - name: app_name
      value: "my-nodejs-app"
    - name: environment  
      value: "dev"
    - name: security_level
      value: "STANDARD"
      
  stages:
    # Étape 1: Scan sécurité de base
    - stage:
        name: Security Scan
        identifier: security_scan
        template:
          templateRef: account.source_code_scan
          versionLabel: v0.1
          templateInputs:
            type: SecurityTests
            spec:
              infrastructure:
                type: KubernetesDirect
                spec:
                  connectorRef: org.kubdesktop
                  namespace: harness-ci
              execution:
                steps:
                  - step:
                      identifier: owasp
                      spec:
                        advanced:
                          fail_on_severity: high  # Permissif pour dev
                  - step:
                      identifier: gitleaks
                      spec:
                        advanced:
                          fail_on_severity: medium
    
    # Étape 2: Build et tests
    - stage:
        name: Build and Test
        identifier: build_test
        template:
          templateRef: account.npm_build
          versionLabel: v0.1
          templateInputs:
            type: CI
            spec:
              infrastructure:
                type: KubernetesDirect
                spec:
                  connectorRef: org.kubdesktop
                  namespace: harness-ci
              execution:
                steps:
                  - step:
                      identifier: Install_Dependencies
                      spec:
                        resources:
                          limits:
                            memory: 2Gi
                            cpu: 1000m
    
    # Étape 3: Déploiement direct
    - stage:
        name: Deploy to Dev
        identifier: deploy_dev
        template:
          templateRef: account.deploy_k8s
          versionLabel: v0.1
          templateInputs:
            type: Deployment
            spec:
              environment:
                environmentRef: dev
              service:
                serviceRef: <+input>
```

### Scénario 2: Application Java - Staging avec Gouvernance
**Contexte**: Application critique, tests d'intégration, approbation DevSecOps

```yaml
# pipeline-staging-java.yaml  
pipeline:
  name: "Java App - Staging Deployment"
  identifier: java_staging_pipeline
  
  variables:
    - name: app_name
      value: "enterprise-java-app"
    - name: environment
      value: "staging" 
    - name: security_level
      value: "REINFORCED"
    - name: coverage_threshold
      value: "75"
      
  stages:
    # Étape 1: Scan sécurité renforcé
    - stage:
        name: Enhanced Security Scan
        identifier: enhanced_security_scan
        template:
          templateRef: account.source_code_scan
          versionLabel: v0.2  # Version renforcée
          templateInputs:
            type: SecurityTests
            spec:
              execution:
                steps:
                  - step:
                      identifier: owasp
                      spec:
                        advanced:
                          fail_on_severity: medium  # Plus strict
                          args: "--enableExperimental"
                        resources:
                          limits:
                            memory: 4Gi
                            cpu: 2000m
                  - step:
                      identifier: aquatrivy  
                      spec:
                        advanced:
                          fail_on_severity: medium
                          args: "--security-checks vuln,config"
                  - step:
                      identifier: gitleaks
                      spec:
                        advanced:
                          fail_on_severity: low  # Zéro tolérance
    
    # Étape 2: Build avec couverture stricte
    - stage:
        name: Build with Quality Gates
        identifier: build_quality
        template:
          templateRef: account.maven_build_stage_tplt
          versionLabel: v0.1
          templateInputs:
            type: CI
            spec:
              execution:
                steps:
                  - step:
                      identifier: maven_test
                      spec:
                        command: |
                          mvn clean test jacoco:report -B
                          
                          # Validation couverture
                          COVERAGE=$(mvn jacoco:report -q | grep -oP 'Total.*?([0-9]+)%' | grep -oP '[0-9]+')
                          if [ "$COVERAGE" -lt "<+pipeline.variables.coverage_threshold>" ]; then
                            echo "❌ Coverage $COVERAGE% below threshold <+pipeline.variables.coverage_threshold>%"
                            exit 1
                          fi
                          echo "✅ Coverage: $COVERAGE%"
    
    # Étape 3: Scan d'image avancé
    - stage:
        name: Advanced Image Security
        identifier: advanced_image_scan
        template:
          templateRef: account.images_scan  
          versionLabel: v0.2
          templateInputs:
            type: SecurityTests
            spec:
              execution:
                steps:
                  - step:
                      identifier: trivy_advanced
                      spec:
                        advanced:
                          fail_on_severity: medium
                          args: "--security-checks vuln,config,secret"
                        image:
                          name: <+artifacts.primary.image>
                          tag: <+artifacts.primary.tag>
    
    # Étape 4: Approbation DevSecOps obligatoire
    - stage:
        name: DevSecOps Approval
        identifier: devsecops_approval
        template:
          templateRef: account.manual_approval
          versionLabel: v0.1
          templateInputs:
            type: Approval
            spec:
              execution:
                steps:
                  - step:
                      identifier: security_review
                      spec:
                        approvers:
                          minimumCount: 1
                          userGroups:
                            - org.DevSecOps_Team
                        approvalMessage: |
                          🔒 **DevSecOps Approval Required - Staging**
                          
                          Application: <+pipeline.variables.app_name>
                          Environment: Staging
                          
                          ✅ Security Scans: PASSED
                          ✅ Code Coverage: <+pipeline.variables.coverage_threshold>%+
                          ✅ Quality Gates: PASSED
                          
                          Please review and approve deployment to staging.
    
    # Étape 5: Déploiement avec monitoring
    - stage:
        name: Deploy to Staging
        identifier: deploy_staging
        template:
          templateRef: account.deploy_k8s
          versionLabel: v0.1
          templateInputs:
            type: Deployment
            spec:
              environment:
                environmentRef: staging
```

### Scénario 3: Production Enterprise - Conformité Maximale
**Contexte**: Application critique, conformité réglementaire, processus d'approbation strict

```yaml
# pipeline-production-enterprise.yaml
pipeline:
  name: "Enterprise Production Deployment"
  identifier: enterprise_prod_pipeline
  
  # Utilisation du template enterprise complet
  template:
    templateRef: account.security_hardened_pipeline_template
    versionLabel: v0.1
    templateInputs:
      properties:
        ci:
          codebase:
            connectorRef: account.github_enterprise
            repoName: <+input>
            build: <+input>
      
      # Configuration des variables enterprise
      variables:
        - name: security_level
          value: "ENTERPRISE_HARDENED"
        - name: compliance_frameworks  
          value: "SOX,GDPR,CIS,NIST"
        - name: manual_approval_required
          value: "true"
        - name: environment
          value: "production"
      
      # Personnalisation des stages
      stages:
        # Validation pré-sécurité avec logs détaillés
        - stage:
            name: Pre-Security Validation
            identifier: pre_security_validation
            spec:
              execution:
                steps:
                  - step:
                      name: Enterprise Security Check
                      spec:
                        command: |
                          echo "🏢 ENTERPRISE SECURITY VALIDATION"
                          echo "================================="
                          echo "Application: <+codebase.repoName>"
                          echo "Branch: <+codebase.branch>"
                          echo "Commit: <+codebase.commitSha>"
                          echo "Environment: PRODUCTION"
                          echo "Compliance: SOX, GDPR, CIS, NIST"
                          echo ""
                          echo "⚠️  STRICT SECURITY POLICIES ACTIVE"
                          echo "❌ Zero tolerance for security violations"
                          
        # Scan sécurisé avec version maximale
        - stage:
            name: Maximum Security Scan
            templateInputs:
              spec:
                execution:
                  steps:
                    - step:
                        identifier: owasp
                        spec:
                          advanced:
                            fail_on_severity: low  # Maximum strict
                    - step:
                        identifier: aquatrivy
                        spec:
                          advanced:
                            fail_on_severity: low
                    - step:
                        identifier: gitleaks  
                        spec:
                          advanced:
                            fail_on_severity: low
        
        # Approbation sécurité renforcée
        - stage:
            name: Security Team Approval
            templateInputs:
              spec:
                execution:
                  steps:
                    - step:
                        identifier: security_approval
                        spec:
                          approvers:
                            minimumCount: 2  # Double validation
                            userGroups:
                              - org.Security_Team
                              - org.DevSecOps_Team
                              - org.Compliance_Team
                          approvalMessage: |
                            🚨 **PRODUCTION DEPLOYMENT APPROVAL**
                            ====================================
                            
                            🏢 **Enterprise Security Review**
                            - Application: <+codebase.repoName>
                            - Environment: PRODUCTION
                            - Security Level: ENTERPRISE_HARDENED
                            
                            ✅ **Security Validation Complete**
                            - Source Code Scan: PASSED (v0.3)
                            - Container Scan: PASSED (v0.3)  
                            - Security Policies: ENFORCED
                            - Compliance: SOX ✅ GDPR ✅ CIS ✅ NIST ✅
                            
                            📊 **Risk Assessment**
                            - Security Score: 98/100
                            - Compliance Score: 100/100
                            - Risk Level: MINIMAL
                            
                            🎯 **Production Readiness**
                            - All security controls validated
                            - Compliance requirements met
                            - Emergency rollback plan confirmed
                            
                            **Required: 2 approvals from Security Team**
                          rejectionMessage: |
                            ❌ **PRODUCTION DEPLOYMENT REJECTED**
                            
                            Security requirements not met.
                            Please address findings and resubmit.
                            
                            Contact: security-team@company.com
```

## 🔧 Input Sets Prêts à l'Usage

### Input Set Development
```yaml
# input-sets/development.yaml
inputSet:
  name: "Development Environment"
  identifier: dev_inputs
  
  pipeline:
    variables:
      environment: "dev"
      security_level: "STANDARD" 
      fail_on_severity: "high"
      manual_approval_required: "false"
    
    stages:
      source_code_scan:
        spec:
          execution:
            steps:
              owasp:
                timeout: 15m
                spec:
                  advanced:
                    fail_on_severity: high
              gitleaks:
                spec:
                  advanced:
                    fail_on_severity: medium
      
      npm_build:
        spec:
          execution:
            steps:
              Install_Dependencies:
                spec:
                  resources:
                    limits:
                      memory: 2Gi
                      cpu: 1000m
```

### Input Set Production
```yaml
# input-sets/production.yaml  
inputSet:
  name: "Production Environment"
  identifier: prod_inputs
  
  pipeline:
    variables:
      environment: "production"
      security_level: "ENTERPRISE_HARDENED"
      compliance_frameworks: "SOX,GDPR,CIS,NIST"
      manual_approval_required: "true"
    
    stages:
      security_hardened_pipeline:
        spec:
          variables:
            security_level: "ENTERPRISE_HARDENED"
            manual_approval_required: "true"
          
          stages:
            security_team_approval:
              spec:
                execution:
                  steps:
                    security_approval:
                      spec:
                        approvers:
                          minimumCount: 2
                          userGroups:
                            - org.Security_Team
                            - org.CISO_Team
```

## ⚙️ Configurations de Connecteurs

### Connecteur Kubernetes Sécurisé
```yaml
# connectors/kubernetes-secure.yaml
connector:
  name: "Kubernetes Secure Cluster"
  identifier: "k8s_secure_cluster"
  type: K8sCluster
  spec:
    credential:
      type: InheritFromDelegate
      spec:
        delegateSelectors:
          - security-delegate
    skipValidation: false
    
  # Configuration sécurisée
  governance:
    - policy: "require-service-account"
    - policy: "network-policies-mandatory"
    - policy: "resource-limits-required"
```

### Connecteur Docker Registry Entreprise
```yaml
# connectors/docker-enterprise.yaml
connector:
  name: "Enterprise Docker Registry"
  identifier: "docker_enterprise"
  type: DockerRegistry
  spec:
    dockerRegistryUrl: "registry.company.com"
    credential:
      type: UsernamePassword
      spec:
        username: <+secrets.getValue("docker_username")>
        passwordRef: <+secrets.getValue("docker_password")>
    executeOnDelegate: true
    delegateSelectors:
      - security-delegate
```

## 🎭 Exemples de Notifications

### Notification Succès de Déploiement
```yaml
notification_rules:
  - name: "Production Success"
    pipelineEvents:
      - type: PipelineSuccess
    notificationMethod:
      type: Slack
      spec:
        webhookUrl: <+secrets.getValue("slack_prod_webhook")>
        message: |
          🎉 **PRODUCTION DEPLOYMENT SUCCESSFUL**
          
          ✅ Application: {{pipeline.name}}
          ✅ Environment: Production
          ✅ Version: {{artifacts.primary.tag}}
          ✅ Security Score: 98/100
          
          🔗 [View Pipeline]({{pipeline.executionUrl}})
          🔗 [Application URL](https://{{service.name}}.company.com)
```

### Alerte Sécurité Critique  
```yaml
notification_rules:
  - name: "Security Critical Alert"
    pipelineEvents:
      - type: StageFailed
        forStages:
          - source_code_scan
          - images_scan
    notificationMethod:
      type: Email
      spec:
        recipients:
          - security-team@company.com
          - devsecops-team@company.com
        subject: "🚨 CRITICAL SECURITY FAILURE - {{pipeline.name}}"
        body: |
          CRITICAL SECURITY FAILURE DETECTED
          ==================================
          
          Pipeline: {{pipeline.name}}
          Stage: {{stage.name}}
          Environment: {{pipeline.variables.environment}}
          
          IMMEDIATE ACTION REQUIRED
          ========================
          1. Review security scan results
          2. Block any manual overrides
          3. Notify development team
          4. Create security incident
          
          Pipeline URL: {{pipeline.executionUrl}}
```

## 📚 Templates de Service et Infrastructure

### Service Definition Sécurisé
```yaml
# services/secure-service-template.yaml
service:
  name: "Secure Application Service"
  identifier: secure_app_service
  serviceDefinition:
    type: Kubernetes
    spec:
      manifests:
        - manifest:
            identifier: deployment
            type: K8sManifest
            spec:
              store:
                type: Harness
                spec:
                  files:
                    - account:/templates/secure-deployment.yaml
      artifacts:
        primary:
          primaryArtifactRef: secure_artifact
          sources:
            - identifier: secure_artifact
              type: DockerRegistry
              spec:
                connectorRef: account.docker_enterprise
                imagePath: company/secure-apps/<+service.name>
                tag: <+input>
      variables:
        - name: security_context
          type: String
          value: "restricted"
        - name: resource_limits
          type: String  
          value: "enforced"
```

Cette documentation fournit aux équipes DevSecOps des exemples concrets et immédiatement utilisables pour implémenter une sécurité progressive et adaptée à chaque environnement.