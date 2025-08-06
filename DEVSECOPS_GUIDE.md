# Guide DevSecOps - Templates Harness

Ce guide fournit aux équipes DevSecOps les informations nécessaires pour utiliser efficacement les templates de sécurité Harness dans leur environnement de production.

## 🎯 Vue d'Ensemble

Cette collection de templates implémente une approche de sécurité par couches ("Defense in Depth") avec:
- **3 niveaux de sécurité** (Standard, Renforcé, Enterprise)
- **4 frameworks de conformité** (SOX, GDPR, CIS, NIST)
- **Validation continue** à chaque étape du pipeline
- **Approche Zero-Trust** pour les déploiements production

## 🔒 Matrice de Sécurité par Environnement

| Environnement | Niveau Sécurité | Templates Requis | Approbations | Seuils Vulnérabilités |
|---------------|----------------|------------------|--------------|----------------------|
| **Development** | Standard | `source_code_scan/v0.1`<br/>`npm_build/v0.1` | Aucune | High: ∞<br/>Critical: 5 |
| **Staging** | Renforcé | `source_code_scan/v0.2`<br/>`images_scan/v0.2` | 1 DevSecOps | High: 3<br/>Critical: 1 |
| **Production** | Enterprise | `security_hardened_pipeline_template/v0.1`<br/>`security_policies/v0.1` | 2 Security Team | High: 0<br/>Critical: 0 |

## 🚀 Workflows d'Usage par Scénario

### Scénario 1: Déploiement Standard (Dev/Test)
```yaml
# Utilisation du pipeline npm standard
pipeline:
  name: "App Deployment - Development"
  stages:
    - template: source_code_scan/v0.1
    - template: npm_build/v0.1
    - template: deploy_k8s/v0.1
```

### Scénario 2: Déploiement Sécurisé (Staging)
```yaml
# Pipeline avec sécurité renforcée
pipeline:
  name: "App Deployment - Staging"
  stages:
    - template: source_code_scan/v0.2
    - template: npm_build/v0.1
    - template: images_scan/v0.2
    - template: manual_approval/v0.1
    - template: deploy_k8s/v0.1
```

### Scénario 3: Déploiement Enterprise (Production)
```yaml
# Pipeline enterprise avec conformité complète
pipeline:
  name: "App Deployment - Production"
  template:
    templateRef: security_hardened_pipeline_template
    versionLabel: v0.1
```

## ⚙️ Configuration des Variables de Sécurité

### Variables Globales Requises
```yaml
# Configuration niveau organisation
variables:
  - name: security_level
    value: "ENTERPRISE_HARDENED"  # STANDARD|REINFORCED|ENTERPRISE_HARDENED
  - name: compliance_frameworks
    value: "SOX,GDPR,CIS,NIST"
  - name: fail_on_severity
    value: "medium"  # low|medium|high|critical
```

### Variables par Environnement
```yaml
# Development
environment: dev
fail_on_severity: high
manual_approval_required: false
security_scan_versions: "v0.1"

# Staging  
environment: staging
fail_on_severity: medium
manual_approval_required: true
security_scan_versions: "v0.2"

# Production
environment: production
fail_on_severity: low
manual_approval_required: true
security_scan_versions: "v0.3"
```

## 🛡️ Processus d'Approbation Sécurisée

### Niveau 1: Approbation DevSecOps (Staging)
- **Déclencheur**: Vulnérabilités Medium détectées
- **Approbateurs**: `org.DevSecOps_Team` (1 personne minimum)
- **Critères**: 
  - Scans sécurité terminés avec succès
  - Aucune vulnérabilité Critical
  - Maximum 3 vulnérabilités High

### Niveau 2: Approbation Security Team (Production)
- **Déclencheur**: Déploiement production
- **Approbateurs**: `org.Security_Team` + `org.DevSecOps_Team` (2 personnes minimum)
- **Critères**:
  - Conformité frameworks validée (SOX, GDPR, CIS, NIST)
  - Aucune vulnérabilité High ou Critical
  - Signature des artefacts validée
  - Audit trail complet

## 📊 Monitoring et Alerting

### Métriques de Sécurité Clés
- **MTTR** (Mean Time To Remediation) des vulnérabilités
- **Taux de conformité** par framework
- **Nombre de déploiements bloqués** par niveau de sécurité
- **Score de sécurité** global des applications

### Notifications d'Alerte
```yaml
# Configuration des alertes critiques
notifications:
  critical_vulnerabilities:
    channels: ["slack", "email", "pagerduty"]
    recipients: ["security-team@company.com"]
  
  failed_deployments:
    channels: ["slack", "email"]
    recipients: ["devsecops-team@company.com"]
    
  compliance_violations:
    channels: ["email", "jira"]
    recipients: ["compliance-team@company.com"]
```

## 🔧 Troubleshooting

### Problèmes Fréquents

#### 1. Échec du Scan de Vulnérabilité
**Symptôme**: Pipeline bloqué sur `source_code_scan`
```bash
# Diagnostic
harness pipeline logs --stage source_code_scan

# Solutions
- Vérifier les seuils de sécurité configurés
- Analyser le rapport de vulnérabilités
- Appliquer les correctifs nécessaires
```

#### 2. Échec de l'Approbation Manuelle
**Symptôme**: Timeout sur l'étape d'approbation
```bash
# Diagnostic  
harness approval status --pipeline-id <ID>

# Solutions
- Vérifier les groupes d'approbateurs
- Notifier les équipes concernées
- Escalader si nécessaire
```

#### 3. Échec du Déploiement Kubernetes
**Symptôme**: Erreur lors du déploiement K8s
```bash
# Diagnostic
kubectl describe pods -n <namespace>
harness service logs --service-id <ID>

# Solutions
- Vérifier les permissions du service account
- Valider la configuration des ressources
- Contrôler les network policies
```

## 📚 Templates de Référence

### Input Sets par Environnement

#### Development
```yaml
# input-set-dev.yaml
pipeline:
  variables:
    environment: "dev"
    security_level: "STANDARD"
    fail_on_severity: "high"
  stages:
    source_code_scan:
      spec:
        execution:
          steps:
            owasp:
              spec:
                advanced:
                  fail_on_severity: "high"
```

#### Production
```yaml
# input-set-prod.yaml
pipeline:
  variables:
    environment: "production"
    security_level: "ENTERPRISE_HARDENED"
    fail_on_severity: "medium"
  stages:
    security_hardened_pipeline:
      spec:
        variables:
          manual_approval_required: "true"
          compliance_frameworks: "SOX,GDPR,CIS,NIST"
```

## 🎯 Bonnes Pratiques DevSecOps

### 1. **Shift-Left Security**
- Intégrer les scans dès le développement
- Former les équipes aux pratiques sécurisées
- Automatiser les contrôles de sécurité

### 2. **Gestion des Vulnérabilités**
- Définir des SLA de remediation par criticité
- Maintenir un registre des exceptions
- Effectuer des revues régulières

### 3. **Conformité Continue**
- Auditer régulièrement les configurations
- Documenter les changements de politique
- Maintenir la traçabilité complète

### 4. **Amélioration Continue**
- Analyser les métriques de sécurité
- Ajuster les seuils selon l'évolution des menaces
- Partager les retours d'expérience

## 📞 Support et Escalade

### Contacts d'Urgence
- **Security Team**: security-team@company.com
- **DevSecOps Team**: devsecops-team@company.com
- **Platform Team**: platform-team@company.com

### Procédure d'Escalade
1. **Niveau 1**: DevSecOps Engineer (< 2h)
2. **Niveau 2**: Security Architect (< 4h)
3. **Niveau 3**: CISO (< 8h)

---
*Ce guide est maintenu par l'équipe DevSecOps. Dernière mise à jour: 2025-08*