Pour convertir votre diagramme Mermaid en SVG avec la CLI, voici les étapes :

## 🔧 **Installation de Mermaid CLI**

```bash
# Installation globale
npm install -g @mermaid-js/mermaid-cli

# Vérifier l'installation
mmdc --version
```

## 📄 **Créer le fichier source**

D'abord, copiez le code Mermaid dans un fichier `.mmd` :

```bash
# Créer le fichier
touch sfdx-hardis-architecture.mmd
```

Puis collez le contenu du diagramme dans `sfdx-hardis-architecture.mmd` (le code qui commence par `graph TB`).

## 🎨 **Conversion en SVG**

```bash
# Conversion basique
mmdc -i sfdx-hardis-architecture.mmd -o sfdx-hardis-architecture.svg

# Avec options avancées
mmdc -i sfdx-hardis-architecture.mmd -o sfdx-hardis-architecture.svg \
  --theme default \
  --width 1920 \
  --height 1080 \
  --backgroundColor white \
  --configFile mermaid-config.json
```

## ⚙️ **Configuration personnalisée (optionnel)**

Créez `mermaid-config.json` pour plus de contrôle :

```json
{
  "theme": "default",
  "themeVariables": {
    "primaryColor": "#ff6b6b",
    "primaryTextColor": "#333",
    "primaryBorderColor": "#333",
    "lineColor": "#666"
  },
  "flowchart": {
    "nodeSpacing": 50,
    "rankSpacing": 50,
    "curve": "basis"
  }
}
```

## 🚀 **Commande finale recommandée**

```bash
mmdc -i sfdx-hardis-architecture.mmd -o sfdx-hardis-architecture.svg \
  --width 1920 \
  --height 1080 \
  --backgroundColor white \
  --theme default
```

Le fichier SVG généré sera parfait pour vos présentations, documentation, ou intégration web ! 📊
