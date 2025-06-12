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

## Configuration GitHub Actions

### 7. Workflow de déploiement (.github/workflows/deploy.yml)

```yaml
name: SFDX Hardis Deployment

on:
  push:
    branches: [integration, uat]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install SFDX CLI and Hardis
      run: |
        npm install -g @salesforce/cli
        npm install -g sfdx-hardis
        
    - name: Authenticate to Salesforce
      run: |
        if [[ ${{ github.ref }} == "refs/heads/integration" ]]; then
          echo "${{ secrets.SFDX_AUTH_URL_INT }}" > auth_url.txt
          sfdx auth:sfdxurl:store -f auth_url.txt -a integration
        elif [[ ${{ github.ref }} == "refs/heads/uat" ]]; then
          echo "${{ secrets.SFDX_AUTH_URL_UAT }}" > auth_url.txt
          sfdx auth:sfdxurl:store -f auth_url.txt -a uat
        fi
        
    - name: Deploy with SFDX Hardis
      run: |
        sfdx hardis:org:deploy:sources:dx
        
    - name: Run Tests
      run: |
        sfdx hardis:org:test:run
```

## Configuration des secrets GitHub

### 8. Récupération des URL d'authentification

```bash
# Pour la sandbox INT
sfdx force:org:display --verbose --targetusername int-sandbox

# Pour la sandbox UAT
sfdx force:org:display --verbose --targetusername uat-sandbox
```

### 9. Configuration dans GitHub Enterprise Cloud

Dans votre repository GitHub Enterprise :
1. Aller dans Settings > Secrets and variables > Actions
2. Ajouter les secrets suivants :
   - `SFDX_AUTH_URL_INT` : URL d'auth de la sandbox INT
   - `SFDX_AUTH_URL_UAT` : URL d'auth de la sandbox UAT

## Commandes utiles SFDX Hardis

### Déploiement manuel
```bash
# Déployer vers INT
sfdx hardis:org:deploy:sources:dx --targetusername int-sandbox

# Déployer vers UAT
sfdx hardis:org:deploy:sources:dx --targetusername uat-sandbox

# Monitoring des environnements
sfdx hardis:org:monitor:all

# Génération de documentation
sfdx hardis:doc:generate
```

### Gestion des conflits
```bash
# Récupérer les métadonnées depuis l'org
sfdx hardis:org:retrieve:sources:dx

# Comparer les environnements
sfdx hardis:org:compare:source
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
