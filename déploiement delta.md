Vous touchez là à un vrai défi des déploiements Salesforce à grande échelle. Voici plusieurs stratégies pour gérer ces contraintes :

## Stratégies pour contourner la limite des 10 000 composants

**1. Déploiement par lots séquentiels**
- Divisez vos 20 000 artefacts en plusieurs packages logiques de <10 000 composants
- Déployez en respectant les dépendances (métadonnées de base → configuration → code)
- Utilisez des scripts pour automatiser la séquence

**2. Approche modulaire par domaine métier**
- Segmentez par modules fonctionnels (Sales, Service, Marketing, etc.)
- Créez des packages indépendants avec leurs propres cycles de déploiement
- Permet une meilleure maintenabilité à long terme

## Outils recommandés

**SFDX/SF CLI avec des scripts custom :**
```bash
# Exemple de déploiement par lots
sf project deploy start --source-dir force-app/main/core
sf project deploy start --source-dir force-app/main/sales
sf project deploy start --source-dir force-app/main/service
```

**Gearset ou Copado :** Excellent pour gérer les déploiements delta intelligents et la gestion des profiles/permission sets

**Vlocity Build Tool / Salesforce CLI :** Pour les déploiements Industry Cloud

## Gestion des Profiles et Permission Sets

**1. Stratégie "Profile minimal + Permission Sets"**
- Gardez les profiles avec permissions minimales
- Utilisez les Permission Sets pour les permissions spécifiques
- Plus facile à gérer en delta

**2. Déploiement sélectif des permissions**
```bash
# Déployer seulement les nouveaux permission sets
sf project deploy start --metadata "PermissionSet:NewPermSet1,NewPermSet2"
```

**3. Post-deployment scripts**
- Scripts Apex pour assigner automatiquement les permissions
- Évite les conflits de profiles en production

## Stratégie de déploiement delta optimisée

**1. Analyse des dépendances**
- Utilisez `sf project list metadata` pour mapper les dépendances
- Déployez dans l'ordre : Schema → Configuration → Code → Permissions

**2. Validation préalable**
```bash
# Validation sans déploiement
sf project deploy validate --source-dir force-app --test-level RunLocalTests
```

**3. Déploiement incrémental intelligent**
- Utilisez Git pour identifier les changements réels
- Ne déployez que les composants modifiés + leurs dépendances

Voulez-vous que je détaille une de ces approches ou que nous explorions votre architecture spécifique pour une recommandation plus ciblée ?
