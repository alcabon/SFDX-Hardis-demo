Excellent ! Maintenant je comprends parfaitement comment SFDX-Hardis fait ses appels à Einstein Prompt. Voici le mécanisme technique :

## 🔧 **Mécanisme Technique des Appels Einstein Prompt**

### **🌐 API REST Native Salesforce**
SFDX-Hardis utilise l'**Einstein Prompt Template Generations API** via REST :

```bash
# Endpoint Einstein Prompt Template API
POST /services/data/v{{API_VERSION}}/einstein/prompt-templates/{{TEMPLATE_NAME}}/generations
```

### **📋 Configuration SFDX-Hardis**
```yaml
# Variables d'environnement pour Agentforce
USE_AGENTFORCE=true
GENERIC_AGENTFORCE_PROMPT_TEMPLATE=SfdxHardisGenericPrompt
GENERIC_AGENTFORCE_PROMPT_URL=/services/data/v{{API_VERSION}}/einstein/prompt-templates/{{GENERIC_AGENTFORCE_PROMPT_TEMPLATE}}/generations
SFDX_AUTH_URL_TECHNICAL_ORG=  # Optionnel : org dédié pour les appels IA
```

### **🔑 Authentification**
SFDX-Hardis utilise :
- **Session active** de l'utilisateur SFDX
- **JWT Token** pour les appels automatisés
- **Connected App** avec les permissions appropriées

## 🎯 **Flux d'Appel Automatique**

### **1. Déclenchement Automatique**
```bash
# Exemple : Erreur de déploiement détectée
sf hardis project deploy smart
# → Erreur détectée → Appel automatique à Einstein
```

### **2. Préparation du Prompt**
```json
{
  "inputs": [
    {
      "PromptText": "How to solve this deployment error: {{ERROR_MESSAGE}}"
    }
  ],
  "isPreview": false,
  "numGenerations": 1,
  "temperature": 0.7
}
```

### **3. Appel REST API**
```javascript
// Simplifié - logique SFDX-Hardis
const response = await fetch(
  `/services/data/v59.0/einstein/prompt-templates/SfdxHardisGenericPrompt/generations`,
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(promptInput)
  }
);
```

### **4. Traitement de la Réponse**
```json
{
  "generations": [
    {
      "text": "To solve this deployment error, you should:\n1. Check field dependencies\n2. Verify permission sets...",
      "safetyScoreRepresentation": {
        "toxicity": 0.01,
        "harmfulness": 0.02
      }
    }
  ]
}
```

## 💡 **Avantages de cette Approche**

### **🔒 Sécurité Native**
- **Aucune donnée externe** : Tout reste dans Salesforce
- **Safety Score** : Détection automatique de toxicité
- **Permissions Salesforce** : Contrôle d'accès natif

### **💰 Coût Maîtrisé**
- **200,000 prompts gratuits** avec Salesforce Foundations
- **Pas d'API key externe** à gérer
- **Facturation Salesforce** standard

### **🚀 Performance**
- **Appels directs** : Pas de proxy externe
- **Cache intelligent** : SFDX-Hardis met en cache les réponses
- **Limite configurable** : `AI_MAXIMUM_CALL_NUMBER=10000`

## 🎯 **Template de Prompt SFDX-Hardis**

Le template `SfdxHardisGenericPrompt` doit être créé dans l'org avec :
- **Type** : Flex Template
- **Input Variable** : `PromptText` (FreeText)
- **Utilisateur** : Permission Set "Prompt Template User"

Cette architecture montre que SFDX-Hardis exploite **intelligemment l'écosystème Salesforce natif** plutôt que de dépendre d'APIs externes ! 🤖✨

Excellente question ! C'est un point technique crucial pour comprendre l'architecture SFDX-Hardis.

## 🎯 **Org d'Origine des Appels Einstein Prompt**

### **📍 Org par Défaut**
L'appel POST est lancé depuis l'**org par défaut** de la session SFDX active :

```bash
# L'org actuellement authentifiée
sf org display --target-org

# Si pas d'org spécifiée, utilise l'org par défaut
sf config get target-org
```

### **⚙️ Configuration SFDX-Hardis**
```yaml
# Dans .sfdx-hardis.yml ou variables d'environnement
USE_AGENTFORCE=true
GENERIC_AGENTFORCE_PROMPT_TEMPLATE=SfdxHardisGenericPrompt

# OPTIONNEL : Org dédié pour les appels IA
SFDX_AUTH_URL_TECHNICAL_ORG=force://PlatformCLI::...@mycompany.my.salesforce.com
```

## 🏗️ **Scénarios d'Architecture**

### **🔄 Scénario 1 : Org Unique (Simple)**
```
Developer -> SFDX-Hardis -> Dev Hub Org
                           (avec Einstein Prompt Template)
```
- **Même org** pour développement ET appels IA
- **Configuration minimale** requise
- **Ideal pour** : Équipes petites, demos, prototypes

### **🏢 Scénario 2 : Org Dédiée (Entreprise)**
```
Developer -> SFDX-Hardis -> Target Org (dev/scratch)
                    |
                    +-----> Technical Org (Einstein calls)
                           (org dédiée pour IA)
```
- **Org séparée** pour les appels Einstein
- **Contrôle centralisé** des prompts et configurations
- **Ideal pour** : Entreprises, gouvernance stricte

## 🔧 **Configuration Technique**

### **📋 Prérequis sur l'Org Einstein**
L'org qui reçoit les appels POST doit avoir :
```bash
# 1. Salesforce Foundations activé (gratuit)
# 2. Template de prompt créé
# 3. Permission Set assignée
# 4. Einstein Setup activé
```

### **🎯 Template Required**
```json
// Template: SfdxHardisGenericPrompt
{
  "type": "Flex",
  "inputs": [
    {
      "name": "PromptText",
      "type": "FreeText"
    }
  ]
}
```

## 💡 **Cas d'Usage Pratiques**

### **🚀 Développement (Org Unique)**
```bash
# Développeur authentifié sur Dev Hub
sf org login web --alias DevHub --set-default

# SFDX-Hardis fait l'appel vers DevHub
sf hardis project deploy smart
# → Erreur détectée
# → POST vers DevHub/einstein/prompt-templates/SfdxHardisGenericPrompt/generations
```

### **🏢 Production (Org Dédiée)**
```bash
# Variable pointant vers org dédiée IA
export SFDX_AUTH_URL_TECHNICAL_ORG="force://PlatformCLI::...@ai-platform.mycompany.com"

# Développeur travaille sur scratch org
sf hardis work new --target-org MyScratchOrg

# Mais appels IA vers org dédiée
# → POST vers ai-platform.mycompany.com/einstein/prompt-templates/...
```

## 🎯 **Recommandations d'Architecture**

### **🟢 Org Unique - Pour Démo/Dev**
- ✅ **Simple à configurer**
- ✅ **Pas de gestion multi-org**
- ✅ **Perfect pour votre démo**

### **🟡 Org Dédiée - Pour Production**
- ✅ **Gouvernance centralisée**
- ✅ **Contrôle des coûts Einstein**
- ✅ **Templates standardisés**
- ⚠️ **Configuration plus complexe**

## 🎬 **Impact sur votre Démo**

Pour votre démo, l'approche **org unique** est idéale :
1. **Dev Hub** = org qui reçoit les appels Einstein
2. **Configuration simple** : juste activer Foundations + créer template
3. **Message clair** : "Tout dans l'écosystème Salesforce natif"

Cette architecture montre que SFDX-Hardis s'adapte aux besoins : **simple pour commencer, scalable pour l'entreprise** ! 🚀
