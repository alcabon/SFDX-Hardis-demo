# Configuration Connected Apps pour GitHub Actions CI/CD

## Option 1 : SFDX Auth URL (Démarrage rapide)

### Avantages
- ✅ Configuration immédiate (5 minutes)
- ✅ Utilise la Connected App SFDX intégrée
- ✅ Parfait pour démarrer rapidement

### Inconvénients
- ⚠️ Moins sécurisé (URL contient des tokens)
- ⚠️ Tokens peuvent expirer
- ⚠️ Difficile à faire tourner (rotation)

### Configuration actuelle (déjà en place)
```bash
# Récupérer les URLs d'auth (déjà fait)
sfdx org:display --verbose --targetusername int-sandbox
sfdx org:display --verbose --targetusername uat-sandbox

# Les stocker dans GitHub Secrets
gh secret set SFDX_AUTH_URL_INT --body "force://PlatformCLI::..."
gh secret set SFDX_AUTH_URL_UAT --body "force://PlatformCLI::..."
```

## Option 2 : Connected Apps dédiées avec SFDX Hardis (Recommandée en production)

### Avantages
- ✅ Plus sécurisé (JWT avec certificats)
- ✅ Contrôle granulaire des permissions
- ✅ Rotation facile des certificats
- ✅ Audit trail complet
- ✅ Configuration automatisée

### Prérequis
```bash
# Installer OpenSSL (nécessaire pour les certificats)
# Windows (Git Bash inclus)
# macOS: brew install openssl
# Linux: apt-get install openssl
```

### Étape 1 : Configuration automatique avec SFDX Hardis

```bash
# Se positionner sur la branche de configuration
git checkout integration  # ou cicd

# Configurer l'authentification pour INT
sf hardis:project:configure:auth --target-org int-sandbox

# Configurer l'authentification pour UAT
sf hardis:project:configure:auth --target-org uat-sandbox

# Si vous utilisez des scratch orgs (DevHub)
sf hardis:project:configure:auth --devhub
```

### Étape 2 : Que fait cette commande automatiquement

La commande `sf hardis:project:configure:auth` va :

1. **Générer un certificat auto-signé**
   ```
   server.key (clé privée)
   server.crt (certificat public)
   ```

2. **Créer une Connected App dans Salesforce** avec :
   - OAuth settings activés
   - JWT Bearer Token Flow activé
   - Certificat public uploadé
   - Permissions appropriées

3. **Mettre à jour `.sfdx-hardis.yml`**
   ```yaml
   branches:
     integration:
       sfdxAuthUrl: "jwt"
       instanceUrl: "https://test.salesforce.com"
       isProduction: false
       
     uat:
       sfdxAuthUrl: "jwt"
       instanceUrl: "https://test.salesforce.com"
       isProduction: false
   ```

4. **Créer les fichiers de configuration**
   - `.sfdx-hardis/<env>.key.encrypt` (clé privée chiffrée)
   - Variables d'environnement pour GitHub

### Étape 3 : Configuration dans GitHub

Après avoir exécuté les commandes, vous devrez ajouter ces secrets dans GitHub :

```bash
# Variables générées par SFDX Hardis
gh secret set SFDX_CLIENT_ID_INT --body "<consumer-key-from-connected-app>"
gh secret set SFDX_JWT_KEY_INT --body "<private-key-content>"

gh secret set SFDX_CLIENT_ID_UAT --body "<consumer-key-from-connected-app>"
gh secret set SFDX_JWT_KEY_UAT --body "<private-key-content>"

# Token de déchiffrement (généré automatiquement)
gh secret set SFDX_HARDIS_DECRYPT_KEY --body "<generated-decrypt-key>"
```

### Étape 4 : Mise à jour des workflows GitHub Actions

```yaml
# .github/workflows/deploy-integration-jwt.yml
name: Deploy to Integration (JWT)

on:
  push:
    branches: [integration]

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
        
    - name: Install dependencies
      run: |
        npm install -g @salesforce/cli
        npm install -g sfdx-hardis
        
    - name: Authenticate with JWT
      run: |
        # Créer le fichier de clé privée temporaire
        echo "${{ secrets.SFDX_JWT_KEY_INT }}" > server.key
        
        # Authentification JWT
        sfdx auth:jwt:grant \
          --clientid "${{ secrets.SFDX_CLIENT_ID_INT }}" \
          --jwtkeyfile server.key \
          --username "${{ secrets.SFDX_USERNAME_INT }}" \
          --instanceurl "https://test.salesforce.com" \
          --alias int-sandbox
          
        # Nettoyer le fichier de clé
        rm server.key
        
    - name: Deploy with SFDX Hardis
      run: |
        sfdx hardis:org:deploy:sources:dx -u int-sandbox
        
    - name: Run tests
      run: |
        sfdx hardis:org:test:run -u int-sandbox
```

## Option 3 : Connected App manuelle (si automatique échoue)

Si la configuration automatique échoue, vous pouvez créer manuellement :

### Dans chaque sandbox (INT et UAT) :

1. **Aller dans Setup > App Manager**
2. **New Connected App** avec :
   - Connected App Name: `CI-CD-App-INT` (ou UAT)
   - API Name: `CI_CD_App_INT`
   - Contact Email: votre email

3. **OAuth Settings** :
   - ✅ Enable OAuth Settings
   - Callback URL: `http://localhost:1717/OauthRedirect`
   - ✅ Use digital signatures
   - Uploader votre certificat (.crt)
   - Selected OAuth Scopes:
     - Access and manage your data (api)
     - Access your basic information (id, profile, email, address, phone)
     - Perform requests on your behalf at any time (refresh_token, offline_access)

4. **Après création** :
   - Edit Policies > Permitted Users: Admin approved users are pre-authorized
   - IP Relaxation: Relax IP restrictions
   - Refresh Token Policy: Refresh token is valid until revoked

## Comparaison des approches

| Critère | SFDX Auth URL | Connected App Auto | Connected App Manuel |
|---------|---------------|-------------------|---------------------|
| **Temps setup** | 5 min | 15 min | 30 min |
| **Sécurité** | Moyenne | Élevée | Élevée |
| **Maintenance** | Difficile | Facile | Moyenne |
| **Rotation tokens** | Manuelle | Automatique | Manuelle |
| **Audit** | Limité | Complet | Complet |
| **Recommended pour** | Dev/Test | Production | Cas spéciaux |

## Recommandation de migration

### Phase 1 : Démarrage (actuel)
- ✅ Utiliser SFDX Auth URL pour démarrer rapidement
- ✅ Tester les workflows et processus

### Phase 2 : Production
- 🔄 Migrer vers Connected Apps avec JWT
- 🔄 Utiliser `sf hardis:project:configure:auth`
- 🔄 Mettre à jour les workflows GitHub

### Phase 3 : Optimisation
- 🔧 Rotation régulière des certificats
- 🔧 Monitoring des connexions
- 🔧 Audit des accès

## Commandes de maintenance

```bash
# Renouveler les certificats (tous les 6-12 mois)
sf hardis:project:configure:auth --target-org int-sandbox --renew-cert

# Vérifier l'authentification
sf hardis:auth:check --target-org int-sandbox

# Lister les Connected Apps
sf data:query --query "SELECT Id, Name, ConsumerKey FROM ConnectedApplication"

# Monitoring des connexions
sf hardis:org:monitor:limits --target-org int-sandbox
```

## Troubleshooting

### Erreur "Invalid JWT"
- Vérifier que l'horloge du serveur CI est synchronisée
- Vérifier que le Consumer Key est correct
- Vérifier que le certificat n'a pas expiré

### Erreur "User not authorized"
- Vérifier les Permitted Users dans la Connected App
- Vérifier les Profile/Permission Set assignments

### Erreur lors de la création automatique
- Vérifier que vous avez les permissions "Manage Connected Apps"
- Vérifier qu'OpenSSL est installé
- Relancer avec `--debug` pour plus de détails
