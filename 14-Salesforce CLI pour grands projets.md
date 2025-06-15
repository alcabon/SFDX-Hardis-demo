# Guide complet sur la gestion des métadonnées Salesforce CLI pour grands projets

## Clarification importante sur la commande "sf project list metadata"

**La commande "sf project list metadata" n'existe pas dans l'écosystème Salesforce CLI actuel.** Cette commande semble être une confusion avec les véritables commandes disponibles. Les commandes appropriées pour la gestion des métadonnées dans le contexte des 20000 artefacts et des contraintes de 10000 composants sont différentes et plus sophistiquées.

## Commandes alternatives réelles pour l'analyse des métadonnées

### Génération de manifestes et inventaire des métadonnées

**Commande principale :**
```bash
# Génération d'un manifeste complet depuis l'org
sf project generate manifest --from-org --output-dir manifest/

# Alternative avec alias d'org spécifique
sf project generate manifest --from-org --target-org production --output-dir manifest/
```

**Récupération sélective des métadonnées :**
```bash
# Récupération par type de métadonnées
sf project retrieve start --metadata CustomObject,CustomField,ApexClass

# Récupération avec manifeste
sf project retrieve start --manifest manifest/package.xml

# Récupération avec sortie JSON pour analyse
sf project retrieve start --metadata CustomObject --json > metadata_analysis.json
```

### Analyse approfondie des dépendances

**Plugin Dependencies CLI (essentiel pour vos besoins) :**
```bash
# Installation du plugin
npm install dependencies-cli --global

# Analyse des dépendances au niveau composant
sf dependency:component:report -u production --json

# Génération de graphiques de dépendances
sf dependency:component:report -u production -r dot | tee dependency_graph.dot

# Identification des composants partagés
sf dependency:component:componentizer -u production
```

## Stratégies pratiques pour 20000 artefacts avec contrainte 10000 composants

### Approche par batches intelligentes

**Script de segmentation automatisée :**
```python
#!/usr/bin/env python3
import json
import subprocess
import math

def analyze_and_batch_metadata():
    # Génération du manifeste complet
    result = subprocess.run(['sf', 'project', 'generate', 'manifest', 
                           '--from-org', '--output-dir', 'temp_manifest'], 
                          capture_output=True, text=True)
    
    # Analyse du package.xml généré
    with open('temp_manifest/package.xml', 'r') as f:
        package_content = f.read()
    
    # Calcul du nombre de composants par type
    # (logique de parsing XML simplifiée)
    component_counts = analyze_package_xml(package_content)
    
    # Création de batches respectant la limite de 10000
    batches = create_deployment_batches(component_counts, max_batch_size=9500)
    
    return batches

def create_deployment_batches(components, max_batch_size=9500):
    """Créer des batches de déploiement optimisés"""
    batches = []
    current_batch = []
    current_count = 0
    
    # Priorisation : objets d'abord, puis champs, puis logique métier
    priority_order = ['CustomObject', 'CustomField', 'ApexClass', 'Flow', 
                     'Layout', 'PermissionSet', 'Profile']
    
    for comp_type in priority_order:
        if comp_type in components:
            type_components = components[comp_type]
            
            if current_count + len(type_components) > max_batch_size:
                # Finaliser le batch actuel
                if current_batch:
                    batches.append(current_batch)
                    current_batch = []
                    current_count = 0
            
            current_batch.append({
                'type': comp_type,
                'members': type_components
            })
            current_count += len(type_components)
    
    # Ajouter le dernier batch
    if current_batch:
        batches.append(current_batch)
    
    return batches
```

### Déploiement par phases pour projets complexes

**Phase 1 : Infrastructure de base**
```bash
# Déploiement des objets personnalisés et champs
sf project deploy start --metadata CustomObject --wait 20
sf project deploy start --metadata CustomField --wait 15
```

**Phase 2 : Logique métier**
```bash
# Déploiement du code Apex et des flux
sf project deploy start --metadata ApexClass,ApexTrigger --wait 30
sf project deploy start --metadata Flow --wait 25
```

**Phase 3 : Interface utilisateur**
```bash
# Déploiement des layouts et composants UI
sf project deploy start --metadata Layout,FlexiPage --wait 20
sf project deploy start --metadata LightningComponentBundle --wait 15
```

**Phase 4 : Sécurité et permissions**
```bash
# Déploiement des profils et permission sets
sf project deploy start --metadata PermissionSet --wait 15
sf project deploy start --metadata Profile --wait 10
```

## Architecture basée sur les packages pour éviter les limites

### Création de packages modulaires

**Setup des packages par domaine fonctionnel :**
```bash
# Package pour le modèle de données
sf package create --name DataModel --type Unlocked --path force-app/data --target-dev-hub production

# Package pour la logique métier
sf package create --name BusinessLogic --type Unlocked --path force-app/logic --target-dev-hub production

# Package pour l'interface utilisateur
sf package create --name UserInterface --type Unlocked --path force-app/ui --target-dev-hub production

# Package pour les intégrations
sf package create --name Integrations --type Unlocked --path force-app/integrations --target-dev-hub production
```

**Versioning et déploiement des packages :**
```bash
# Création des versions
sf package version create --package DataModel --wait 10 --target-dev-hub production
sf package version create --package BusinessLogic --wait 10 --target-dev-hub production

# Installation ordonnée dans l'org cible
sf package install --package DataModel@1.0.0-1 --target-org staging --wait 10
sf package install --package BusinessLogic@1.0.0-1 --target-org staging --wait 10
```

## Script d'automatisation pour CI/CD avec gestion des contraintes

**Pipeline GitHub Actions optimisé :**
```yaml
name: Large Scale Salesforce Deployment
on:
  push:
    branches: [main]

jobs:
  analyze-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Salesforce CLI
        run: |
          npm install @salesforce/cli --global
          npm install dependencies-cli --global
        
      - name: Authenticate to org
        run: |
          sf org login jwt --username ${{ secrets.SF_USERNAME }} \
                          --jwt-key-file server.key \
                          --alias production
        
      - name: Analyze metadata and dependencies
        run: |
          # Génération du manifeste complet
          sf project generate manifest --from-org --target-org production --output-dir analysis/
          
          # Comptage des composants
          COMPONENT_COUNT=$(python3 scripts/count_components.py analysis/package.xml)
          echo "Total components: $COMPONENT_COUNT"
          
          if [ $COMPONENT_COUNT -gt 9000 ]; then
            echo "Large deployment detected. Using intelligent batching."
            python3 scripts/create_deployment_batches.py
            
            # Déploiement par batches
            for batch in batches/*.xml; do
              echo "Deploying batch: $batch"
              sf project deploy start --manifest $batch --target-org production --wait 30
              
              # Vérification du succès avant le batch suivant
              if [ $? -ne 0 ]; then
                echo "Batch deployment failed. Stopping pipeline."
                exit 1
              fi
            done
          else
            # Déploiement standard pour les petits changements
            sf project deploy start --manifest analysis/package.xml --target-org production
          fi
        
      - name: Post-deployment verification
        run: |
          # Vérification de l'intégrité après déploiement
          sf dependency:component:report -u production --json > post_deploy_analysis.json
          python3 scripts/verify_deployment_integrity.py
```

## Monitoring et optimisation des performances

### Script de surveillance des déploiements

**Monitoring en temps réel :**
```javascript
// monitoring_script.js
const { execSync } = require('child_process');
const fs = require('fs');

class SalesforceDeploymentMonitor {
    constructor(orgAlias) {
        this.orgAlias = orgAlias;
        this.maxComponentLimit = 9500; // Buffer sous la limite de 10000
        this.maxSizeLimit = 35; // MB avec marge de sécurité
    }
    
    async monitorDeployment(manifestPath) {
        console.log('Starting deployment monitoring...');
        
        const analysis = this.analyzeManifest(manifestPath);
        
        if (analysis.componentCount > this.maxComponentLimit) {
            console.log(`⚠️  Component limit exceeded: ${analysis.componentCount}`);
            return this.executeBatchDeployment(manifestPath);
        }
        
        if (analysis.estimatedSize > this.maxSizeLimit) {
            console.log(`⚠️  Size limit exceeded: ${analysis.estimatedSize}MB`);
            return this.executeBatchDeployment(manifestPath);
        }
        
        // Déploiement standard
        return this.executeStandardDeployment(manifestPath);
    }
    
    analyzeManifest(manifestPath) {
        // Analyse du manifeste pour estimation de taille et comptage
        const manifestContent = fs.readFileSync(manifestPath, 'utf8');
        const componentCount = this.countComponents(manifestContent);
        const estimatedSize = this.estimateDeploymentSize(manifestContent);
        
        return {
            componentCount,
            estimatedSize,
            recommendedStrategy: componentCount > 5000 ? 'batch' : 'standard'
        };
    }
    
    executeBatchDeployment(manifestPath) {
        console.log('🔄 Executing intelligent batch deployment...');
        
        // Création des batches optimisés
        const batches = this.createOptimizedBatches(manifestPath);
        
        batches.forEach((batch, index) => {
            console.log(`📦 Deploying batch ${index + 1}/${batches.length}`);
            
            try {
                const result = execSync(`sf project deploy start --manifest ${batch.path} --target-org ${this.orgAlias} --wait 30`, 
                                      { encoding: 'utf8' });
                console.log(`✅ Batch ${index + 1} deployed successfully`);
                
                // Pause entre les batches pour éviter la surcharge
                if (index < batches.length - 1) {
                    console.log('⏳ Waiting 30 seconds before next batch...');
                    execSync('sleep 30');
                }
            } catch (error) {
                console.error(`❌ Batch ${index + 1} failed:`, error.message);
                throw error;
            }
        });
        
        console.log('🎉 All batches deployed successfully');
    }
}

// Usage
const monitor = new SalesforceDeploymentMonitor('production');
monitor.monitorDeployment('manifest/package.xml').catch(console.error);
```

## Différences avec les anciennes commandes SFDX

### Migration des commandes obsolètes

**Anciennes commandes SFDX (dépréciées depuis novembre 2024) :**
- `sfdx force:source:retrieve` → `sf project retrieve start`
- `sfdx force:source:deploy` → `sf project deploy start`
- `sfdx force:source:manifest:create` → `sf project generate manifest`

**Nouvelles fonctionnalités SF CLI v2 :**
- Support natif pour les déploiements par batches
- Intégration améliorée avec les packages
- Validation préalable avec `sf project deploy validate`
- Prévisualisation avec `sf project deploy preview`

## Meilleures pratiques pour les architectures de 20000+ composants

### Gouvernance des métadonnées

1. **Segmentation par domaines fonctionnels** : Divisez vos 20000 artefacts en packages logiques de 2000-3000 composants chacun

2. **Pipeline de déploiement en cascade** : 
   - Développement → Intégration (batches de 5000)
   - Intégration → Staging (batches de 7500)  
   - Staging → Production (batches de 9500)

3. **Monitoring continu** : Surveillez les limites d'API et les performances avec des dashboards personnalisés

4. **Stratégie de rollback** : Maintenez des versions de packages pour permettre des retours en arrière rapides

### Architecture recommandée pour 20000+ composants

```
├── core-platform/           (Package 1: 3000 composants)
│   ├── objects/
│   ├── fields/
│   └── core-workflows/
├── business-logic/          (Package 2: 4000 composants)
│   ├── apex-classes/
│   ├── triggers/
│   └── flows/
├── user-interface/          (Package 3: 5000 composants)
│   ├── lightning-components/
│   ├── pages/
│   └── layouts/
├── integrations/           (Package 4: 3000 composants)
│   ├── external-services/
│   ├── apis/
│   └── connectors/
└── security-config/        (Package 5: 5000 composants)
    ├── profiles/
    ├── permission-sets/
    └── sharing-rules/
```

Cette approche modulaire permet de respecter les contraintes de 10000 composants par déploiement tout en maintenant une architecture cohérente et maintenable pour vos 20000 artefacts.

## Conclusion

Bien que la commande "sf project list metadata" n'existe pas, l'écosystème Salesforce CLI v2 offre des outils bien plus sophistiqués pour gérer les grands projets. L'approche recommandée combine l'utilisation de `sf project generate manifest`, le plugin `dependencies-cli`, et une architecture basée sur les packages pour surmonter efficacement les contraintes de déploiement tout en maintenant la performance et la fiabilité.
