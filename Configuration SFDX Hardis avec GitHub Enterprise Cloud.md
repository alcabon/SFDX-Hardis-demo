# Configuration SFDX Hardis avec GitHub Enterprise Cloud

## Prérequis

### Outils nécessaires
- Node.js (version 18 ou supérieure)
- Git
- Salesforce CLI (sfdx)
- Accès à GitHub Enterprise Cloud
- Accès aux sandboxes Salesforce (INT et UAT)

### Installation des outils

```bash
# Installation de Salesforce CLI
npm install -g @salesforce/cli

# Installation de SFDX Hardis
npm install -g sfdx-hardis

# Vérification des installations
sfdx --version
sfdx hardis:version
```

## Configuration initiale du projet

### 1. Cloner ou créer le repository GitHub Enterprise

```bash
# Cloner le repository existant
git clone https://github.com/votre-entreprise/votre-projet-salesforce.git
cd votre-projet-salesforce

# Ou créer un nouveau projet
mkdir votre-projet-salesforce
cd votre-projet-salesforce
git init
```

### 2. Initialiser le projet SFDX Hardis

```bash
# Initialiser le projet avec SFDX Hardis
sfdx hardis:project:create --projectname "Votre Projet" --template "scratch"

# Configuration des environnements
sfdx hardis:org:configure:monitoring
```

## Configuration des environnements

### 3. Configuration des sandboxes

#### Connexion à la sandbox INT
```bash
# Authentification à la sandbox INT
sfdx auth:web:login --setalias int-sandbox --instanceurl https://test.salesforce.com

# Définir comme org par défaut pour INT
sfdx config:set defaultusername=int-sandbox

# Vérifier la connexion
sfdx org:display --targetusername int-sandbox
```

#### Connexion à la sandbox UAT
```bash
# Authentification à la sandbox UAT
sfdx auth:web:login --setalias uat-sandbox --instanceurl https://test.salesforce.com

# Vérifier la connexion
sfdx org:display --targetusername uat-sandbox
```

### 4. Configuration du fichier sfdx-project.json

```json
{
  "packageDirectories": [
    {
      "path": "force-app",
      "default": true,
      "package": "VotreProjet",
      "versionName": "ver 1.0",
      "versionNumber": "1.0.0.NEXT"
    }
  ],
  "name": "VotreProjet",
  "namespace": "",
  "sfdcLoginUrl": "https://login.salesforce.com",
  "sourceApiVersion": "59.0",
  "plugins": {
    "sfdx-hardis": {}
  }
}
```

## Configuration des branches Git

### 5. Structure des branches recommandée

```bash
# Création des branches principales
git checkout -b develop
git checkout -b integration
git checkout -b uat
git checkout -b main

# Retour sur develop pour le développement
git checkout develop
```

### 6. Configuration des environnements dans .sfdx-hardis.yml

```yaml
# .sfdx-hardis.yml
environments:
  integration:
    sfdxAuthUrl: "force://..."  # URL d'auth pour INT
    isProduction: false
    branches:
      - integration
    deploymentPlan: "integration"
  
  uat:
    sfdxAuthUrl: "force://..."  # URL d'auth pour UAT
    isProduction: false
    branches:
      - uat
    deploymentPlan: "uat"

monitoring:
  enabled: true
  branches:
    - integration
    - uat
    - main

deployment:
  testLevel: "RunLocalTests"
  checkOnly: false
  waitTime: 10
```

## Configuration GitHub Actions détaillée

### 7. Workflow de déploiement vers INT (.github/workflows/deploy-integration.yml)

```yaml
name: Deploy to Integration

on:
  push:
    branches: [integration]
  workflow_dispatch:

env:
  SFDX_IMPROVED_CODE_COVERAGE: true
  SFDX_USE_PROGRESS_BAR: false

jobs:
  deploy-integration:
    runs-on: ubuntu-latest
    environment: integration
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Install dependencies
      run: |
        npm install -g @salesforce/cli
        npm install -g sfdx-hardis
        npm install
        
    - name: Authenticate to Salesforce INT
      run: |
        echo "${{ secrets.SFDX_AUTH_URL_INT }}" > auth_url.txt
        sfdx auth:sfdxurl:store -f auth_url.txt -a int-sandbox
        rm auth_url.txt
        
    - name: Pre-deployment checks
      run: |
        sfdx hardis:org:diagnose:legacyapi -u int-sandbox
        sfdx hardis:lint:access
        
    - name: Deploy to Integration
      run: |
        sfdx hardis:org:deploy:sources:dx -u int-sandbox --check-only false
        
    - name: Run tests
      run: |
        sfdx hardis:org:test:run -u int-sandbox
        
    - name: Post-deployment monitoring
      run: |
        sfdx hardis:org:monitor:backup -u int-sandbox
        sfdx hardis:doc:generate:deploymentreport
        
    - name: Upload deployment artifacts
      uses: actions/upload-artifact@v4
      if: always()
      with:
        name: integration-deployment-report
        path: |
          hardis-report/
          deployment-report.html
```

### 8. Workflow de déploiement vers UAT (.github/workflows/deploy-uat.yml)

```yaml
name: Deploy to UAT

on:
  push:
    branches: [uat]
  workflow_dispatch:

env:
  SFDX_IMPROVED_CODE_COVERAGE: true
  SFDX_USE_PROGRESS_BAR: false

jobs:
  deploy-uat:
    runs-on: ubuntu-latest
    environment: uat
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Install dependencies
      run: |
        npm install -g @salesforce/cli
        npm install -g sfdx-hardis
        npm install
        
    - name: Authenticate to Salesforce UAT
      run: |
        echo "${{ secrets.SFDX_AUTH_URL_UAT }}" > auth_url.txt
        sfdx auth:sfdxurl:store -f auth_url.txt -a uat-sandbox
        rm auth_url.txt
        
    - name: Pre-deployment validation
      run: |
        sfdx hardis:org:diagnose:legacyapi -u uat-sandbox
        sfdx hardis:lint:access
        sfdx hardis:org:compare:source -u uat-sandbox
        
    - name: Deploy to UAT (Check Only first)
      run: |
        sfdx hardis:org:deploy:sources:dx -u uat-sandbox --check-only true
        
    - name: Deploy to UAT (Real deployment)
      if: success()
      run: |
        sfdx hardis:org:deploy:sources:dx -u uat-sandbox --check-only false
        
    - name: Run comprehensive tests
      run: |
        sfdx hardis:org:test:run -u uat-sandbox --coverage-requirements 85
        
    - name: Generate documentation
      run: |
        sfdx hardis:doc:generate:deploymentreport
        sfdx hardis:doc:generate:changelog
        
    - name: Upload deployment artifacts
      uses: actions/upload-artifact@v4
      if: always()
      with:
        name: uat-deployment-report
        path: |
          hardis-report/
          deployment-report.html
          changelog.md
```

### 9. Workflow de validation PR (.github/workflows/pr-validation.yml)

```yaml
name: PR Validation

on:
  pull_request:
    branches: [integration, uat, main]

jobs:
  validate:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0
        
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Install dependencies
      run: |
        npm install -g @salesforce/cli
        npm install -g sfdx-hardis
        npm install
        
    - name: Lint and validate
      run: |
        sfdx hardis:lint:access
        sfdx scanner:run --target "force-app" --format "table"
        
    - name: Check deployment validity
      run: |
        sfdx hardis:project:deploy:sources:dx --check-only true
        
    - name: Generate PR report
      run: |
        sfdx hardis:doc:generate:changelog --output-file pr-changelog.md
        
    - name: Comment PR
      uses: actions/github-script@v7
      with:
        script: |
          const fs = require('fs');
          if (fs.existsSync('pr-changelog.md')) {
            const changelog = fs.readFileSync('pr-changelog.md', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## 📋 Changements détectés\n\n${changelog}`
            });
          }
```

### 10. Workflow de monitoring quotidien (.github/workflows/daily-monitoring.yml)

```yaml
name: Daily Monitoring

on:
  schedule:
    - cron: '0 8 * * 1-5'  # Tous les jours ouvrables à 8h
  workflow_dispatch:

jobs:
  monitor-environments:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        environment: [integration, uat]
        
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        
    - name: Install dependencies
      run: |
        npm install -g @salesforce/cli
        npm install -g sfdx-hardis
        
    - name: Authenticate
      run: |
        if [ "${{ matrix.environment }}" == "integration" ]; then
          echo "${{ secrets.SFDX_AUTH_URL_INT }}" > auth_url.txt
          sfdx auth:sfdxurl:store -f auth_url.txt -a int-sandbox
          TARGET_ORG="int-sandbox"
        else
          echo "${{ secrets.SFDX_AUTH_URL_UAT }}" > auth_url.txt
          sfdx auth:sfdxurl:store -f auth_url.txt -a uat-sandbox
          TARGET_ORG="uat-sandbox"
        fi
        rm auth_url.txt
        echo "TARGET_ORG=$TARGET_ORG" >> $GITHUB_ENV
        
    - name: Run monitoring
      run: |
        sfdx hardis:org:diagnose:legacyapi -u ${{ env.TARGET_ORG }}
        sfdx hardis:org:monitor:backup -u ${{ env.TARGET_ORG }}
        sfdx hardis:org:monitor:limits -u ${{ env.TARGET_ORG }}
        
    - name: Generate monitoring report
      run: |
        sfdx hardis:doc:generate:monitoringreport -u ${{ env.TARGET_ORG }}
        
    - name: Upload monitoring report
      uses: actions/upload-artifact@v4
      with:
        name: monitoring-${{ matrix.environment }}-${{ github.run_number }}
        path: hardis-report/
```

## Configuration avancée GitHub Enterprise

### 11. Configuration des Environments GitHub

Dans votre repository GitHub Enterprise, configurez les environnements :

#### Settings > Environments > Create environment "integration"
- **Deployment branches** : `integration` uniquement
- **Required reviewers** : Optionnel pour INT
- **Wait timer** : 0 minutes
- **Secrets** :
  - `SFDX_AUTH_URL_INT` : URL d'authentification sandbox INT

#### Settings > Environments > Create environment "uat"  
- **Deployment branches** : `uat` uniquement
- **Required reviewers** : Équipe UAT (recommandé)
- **Wait timer** : 5 minutes (optionnel)
- **Secrets** :
  - `SFDX_AUTH_URL_UAT` : URL d'authentification sandbox UAT

### 12. Configuration des Branch Protection Rules

```bash
# Via GitHub CLI (optionnel)
gh api repos/:owner/:repo/branches/integration/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["validate"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":1}' \
  --field restrictions=null

gh api repos/:owner/:repo/branches/uat/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["deploy-integration"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":2}' \
  --field restrictions=null
```

### 13. Workflow de release (.github/workflows/release.yml)

```yaml
name: Create Release

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      version:
        description: 'Version number (e.g., 1.2.3)'
        required: true
        type: string

jobs:
  create-release:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0
        
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        
    - name: Install dependencies
      run: |
        npm install -g @salesforce/cli
        npm install -g sfdx-hardis
        
    - name: Generate release notes
      run: |
        sfdx hardis:doc:generate:changelog --output-file RELEASE_NOTES.md
        sfdx hardis:doc:generate:deploymentreport
        
    - name: Create package version
      run: |
        VERSION=${{ github.event.inputs.version || '1.0.0' }}
        sfdx force:package:version:create \
          --package "VotreProjet" \
          --installation-key-bypass \
          --wait 10 \
          --code-coverage
          
    - name: Create GitHub Release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: v${{ github.event.inputs.version || '1.0.0' }}
        release_name: Release v${{ github.event.inputs.version || '1.0.0' }}
        body_path: RELEASE_NOTES.md
        draft: false
        prerelease: false
```

### 14. Workflow de hotfix (.github/workflows/hotfix.yml)

```yaml
name: Hotfix Deployment

on:
  workflow_dispatch:
    inputs:
      target_environment:
        description: 'Target environment'
        required: true
        type: choice
        options:
        - integration
        - uat
        - production
      hotfix_description:
        description: 'Description of the hotfix'
        required: true
        type: string

jobs:
  hotfix-deploy:
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.target_environment }}
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        
    - name: Install dependencies
      run: |
        npm install -g @salesforce/cli
        npm install -g sfdx-hardis
        
    - name: Authenticate to target environment
      run: |
        if [ "${{ github.event.inputs.target_environment }}" == "integration" ]; then
          echo "${{ secrets.SFDX_AUTH_URL_INT }}" > auth_url.txt
          TARGET_ORG="int-sandbox"
        elif [ "${{ github.event.inputs.target_environment }}" == "uat" ]; then
          echo "${{ secrets.SFDX_AUTH_URL_UAT }}" > auth_url.txt
          TARGET_ORG="uat-sandbox"
        fi
        sfdx auth:sfdxurl:store -f auth_url.txt -a $TARGET_ORG
        rm auth_url.txt
        echo "TARGET_ORG=$TARGET_ORG" >> $GITHUB_ENV
        
    - name: Create hotfix backup
      run: |
        sfdx hardis:org:monitor:backup -u ${{ env.TARGET_ORG }}
        
    - name: Deploy hotfix
      run: |
        sfdx hardis:org:deploy:sources:dx -u ${{ env.TARGET_ORG }} --check-only false
        
    - name: Create hotfix documentation
      run: |
        echo "# Hotfix Deployment Report" > hotfix-report.md
        echo "**Date:** $(date)" >> hotfix-report.md
        echo "**Environment:** ${{ github.event.inputs.target_environment }}" >> hotfix-report.md
        echo "**Description:** ${{ github.event.inputs.hotfix_description }}" >> hotfix-report.md
        echo "**Deployed by:** ${{ github.actor }}" >> hotfix-report.md
        
    - name: Create issue for hotfix tracking
      uses: actions/github-script@v7
      with:
        script: |
          const fs = require('fs');
          const report = fs.readFileSync('hotfix-report.md', 'utf8');
          
          github.rest.issues.create({
            owner: context.repo.owner,
            repo: context.repo.repo,
            title: `🚨 Hotfix deployed to ${{ github.event.inputs.target_environment }}`,
            body: report,
            labels: ['hotfix', '${{ github.event.inputs.target_environment }}']
          });
```

## Commandes utiles pour GitHub Actions

### Déploiement et gestion via CLI
```bash
# Déclencher un déploiement manuel
gh workflow run "Deploy to Integration" --ref integration

# Déclencher un déploiement UAT
gh workflow run "Deploy to UAT" --ref uat

# Déclencher un hotfix
gh workflow run "Hotfix Deployment" \
  --field target_environment=integration \
  --field hotfix_description="Fix critical issue XYZ"

# Voir le statut des workflows
gh run list --workflow="Deploy to Integration"

# Télécharger les artifacts de déploiement
gh run download <run-id> --name integration-deployment-report
```

### Monitoring et maintenance via GitHub Actions
```bash
# Déclencher le monitoring manuel
gh workflow run "Daily Monitoring"

# Voir les logs d'un workflow
gh run view <run-id> --log

# Relancer un workflow échoué
gh run rerun <run-id>
```

### Configuration des secrets (une seule fois)
```bash
# Récupérer les URL d'authentification
sfdx org:display --verbose --targetusername int-sandbox
sfdx org:display --verbose --targetusername uat-sandbox

# Ajouter les secrets via GitHub CLI
gh secret set SFDX_AUTH_URL_INT --body "force://PlatformCLI::..."
gh secret set SFDX_AUTH_URL_UAT --body "force://PlatformCLI::..."

# Définir les secrets au niveau des environnements
gh api repos/:owner/:repo/environments/integration/secrets/SFDX_AUTH_URL_INT \
  --method PUT \
  --field encrypted_value="<encrypted-value>"
```

## Structure de projet recommandée

```
votre-projet-salesforce/
├── .github/
│   ├── workflows/
│   │   ├── deploy-integration.yml
│   │   ├── deploy-uat.yml
│   │   ├── pr-validation.yml
│   │   ├── daily-monitoring.yml
│   │   ├── release.yml
│   │   └── hotfix.yml
│   └── ISSUE_TEMPLATE/
├── force-app/
│   └── main/
│       └── default/
├── config/
│   ├── project-scratch-def.json
│   └── .sfdx-hardis.yml
├── scripts/
│   ├── apex/
│   └── data/
├── .sfdx-hardis.yml
├── sfdx-project.json
├── package.json
└── README.md
```

## Exemple de package.json pour automatisation

```json
{
  "name": "votre-projet-salesforce",
  "version": "1.0.0",
  "scripts": {
    "deploy:int": "sfdx hardis:org:deploy:sources:dx -u int-sandbox",
    "deploy:uat": "sfdx hardis:org:deploy:sources:dx -u uat-sandbox",
    "test:int": "sfdx hardis:org:test:run -u int-sandbox",
    "test:uat": "sfdx hardis:org:test:run -u uat-sandbox",
    "lint": "sfdx hardis:lint:access",
    "monitor": "sfdx hardis:org:monitor:all",
    "backup": "sfdx hardis:org:monitor:backup",
    "docs": "sfdx hardis:doc:generate"
  },
  "devDependencies": {
    "@salesforce/cli": "^2.15.0",
    "sfdx-hardis": "^4.40.0"
  }
}
```

## Workflow de développement recommandé

1. **Développement** : Travailler sur la branche `develop`
2. **Intégration** : Merger vers `integration` → Déploie automatiquement vers sandbox INT
3. **Test UAT** : Merger vers `uat` → Déploie automatiquement vers sandbox UAT
4. **Production** : Créer une PR vers `main` après validation UAT

## Monitoring et maintenance

### Configuration du monitoring
```bash
# Activer le monitoring
sfdx hardis:org:configure:monitoring

# Configurer les notifications
sfdx hardis:org:configure:notifications
```

### Commandes de maintenance
```bash
# Nettoyage des métadonnées obsolètes
sfdx hardis:org:purge:obsolete

# Backup des données importantes
sfdx hardis:org:data:export

# Analyse de la qualité du code
sfdx hardis:lint
```

## Bonnes pratiques

- Toujours tester en INT avant UAT
- Utiliser des feature branches pour les développements
- Maintenir la documentation à jour avec `sfdx hardis:doc:generate`
- Surveiller régulièrement les performances avec le monitoring
- Effectuer des backups réguliers avant les déploiements majeurs
