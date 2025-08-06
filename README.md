# Harness Generic Template Repository

Templates CI/CD sécurisés pour déploiements enterprise avec Harness.io

## 📚 Documentation DevSecOps

### 🎯 Guides Principaux
- **[Guide DevSecOps](DEVSECOPS_GUIDE.md)** - Guide complet d'utilisation pour les équipes DevSecOps
- **[CLAUDE.md](CLAUDE.md)** - Documentation technique pour Claude Code

### 🔒 Sécurité et Conformité  
- **[Matrice de Sécurité](docs/security/SECURITY_MATRIX.md)** - Niveaux de sécurité par environnement et frameworks de conformité
- **[Processus d'Approbation](docs/security/APPROVAL_PROCESSES.md)** - Workflows d'approbation sécurisée et escalade

### 📖 Exemples Pratiques
- **[Exemples d'Usage](docs/examples/USAGE_EXAMPLES.md)** - Configurations types et cas d'usage concrets

## 🚀 Démarrage Rapide

### Pour les Équipes de Développement
```bash
# Utiliser le template standard pour développement
harness-cli pipeline create --template source_code_scan/v0.1
```

### Pour les Équipes DevSecOps  
```bash
# Déployer avec sécurité renforcée (staging)
harness-cli pipeline create --template security_hardened_pipeline_template/v0.1
```

### Pour les Déploiements Production
```bash
# Pipeline enterprise avec conformité complète
harness-cli pipeline create --template security_hardened_pipeline_template/v0.1 \
  --input-set production_inputs
```

## 📋 Templates Disponibles

| Template | Version | Environnement | Sécurité | Description |
|----------|---------|---------------|----------|-------------|
| `npm_build` | v0.1 | Dev/Staging | Standard | Build Node.js avec tests |
| `maven_build_stage_tplt` | v0.1 | Dev/Staging | Standard | Build Java avec couverture |
| `source_code_scan` | v0.1-v0.3 | Tous | Progressive | Scans sécurité du code |
| `images_scan` | v0.1-v0.3 | Tous | Progressive | Scans containers |
| `security_hardened_pipeline_template` | v0.1 | Production | Enterprise | Pipeline sécurisé complet |
| `security_policies` | v0.1 | Production | Enterprise | Gouvernance et conformité |

## 🏢 Architecture Enterprise

### Niveaux de Sécurité
- **Standard**: Développement et tests
- **Renforcé**: Staging et pré-production  
- **Enterprise**: Production et applications critiques

### Frameworks de Conformité
- **SOX** (Sarbanes-Oxley)
- **GDPR** (General Data Protection Regulation)
- **CIS** Controls (Center for Internet Security)
- **NIST** Cybersecurity Framework

## 🔧 Configuration Organisation

```yaml
organization: stf
project: harness_generic_template
git_connector: account.github_enterprise
docker_registry: org.docker_enterprise
k8s_clusters:
  - org.kubdesktop (dev)
  - org.k8s_staging (staging)  
  - org.k8s_production (prod)
```

## 📞 Support

- **Documentation**: Voir les guides dans `/docs`
- **DevSecOps Team**: devsecops-team@company.com
- **Security Team**: security-team@company.com
- **Issues**: Utiliser le système de tickets interne