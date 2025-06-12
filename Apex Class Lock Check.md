
```action
name: Apex Class Lock Check

# DÉCLENCHEUR : Le workflow se lance automatiquement
on:
  pull_request:
    # Seulement si des fichiers dans ce path sont modifiés
    paths:
      - 'force-app/main/default/classes/**'
    # Types d'événements PR qui déclenchent le workflow
    types: [opened, synchronize, reopened]

jobs:
  check-locks:
    runs-on: ubuntu-latest
    
    steps:
      # 1. Récupérer le code du repo
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Pour avoir l'historique complet
      
      # 2. Identifier les fichiers Apex modifiés dans cette PR
      - name: Get modified Apex classes
        id: modified-files
        run: |
          # Récupère la liste des fichiers modifiés dans la PR
          git diff --name-only origin/${{ github.base_ref }}..HEAD \
            | grep "force-app/main/default/classes/.*\.cls$" \
            | sed 's|.*/||' \
            | sed 's|\.cls$||' > modified_classes.txt
          
          echo "Modified classes:"
          cat modified_classes.txt
          
          # Stocke la liste pour les étapes suivantes
          echo "modified_classes=$(cat modified_classes.txt | tr '\n' ' ')" >> $GITHUB_OUTPUT
      
      # 3. Vérifier les issues avec le label "apex-lock"
      - name: Check for locked classes
        id: check-locks
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          # Récupère toutes les issues ouvertes avec le label "apex-lock"
          gh api repos/${{ github.repository }}/issues \
            --jq '.[] | select(.state == "open" and (.labels[] | .name == "apex-lock")) | .title' \
            > locked_classes.txt
          
          echo "Currently locked classes:"
          cat locked_classes.txt
          
          # Compare les classes modifiées avec les classes verrouillées
          conflicts=""
          while IFS= read -r modified_class; do
            if [ -n "$modified_class" ]; then
              if grep -q "$modified_class" locked_classes.txt; then
                conflicts="$conflicts $modified_class"
              fi
            fi
          done < modified_classes.txt
          
          echo "conflicts=$conflicts" >> $GITHUB_OUTPUT
          
          if [ -n "$conflicts" ]; then
            echo "CONFLICT DETECTED: Classes $conflicts are currently locked!"
            exit 1
          else
            echo "No conflicts detected. All modified classes are available."
          fi
      
      # 4. Commenter la PR en cas de conflit (optionnel)
      - name: Comment PR on conflict
        if: failure()
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          conflicts="${{ steps.check-locks.outputs.conflicts }}"
          gh pr comment ${{ github.event.number }} --body "
          ⚠️ **Conflit détecté !**
          
          Les classes Apex suivantes sont actuellement verrouillées : **$conflicts**
          
          Veuillez vérifier les issues avec le label \`apex-lock\` pour plus d'informations.
           "
```

# FONCTIONNEMENT DÉTAILLÉ :

 1. DÉCLENCHEMENT
    - Se lance quand une PR est créée/mise à jour
    - Seulement si elle modifie des fichiers .cls dans force-app/main/default/classes/

 2. DÉTECTION DES FICHIERS MODIFIÉS
    - Compare la branche de la PR avec la branche cible
    - Filtre pour ne garder que les classes Apex (.cls)
    - Extrait juste le nom de la classe (sans path ni extension)

 3. VÉRIFICATION DES LOCKS
    - Utilise l'API GitHub pour récupérer les issues ouvertes
    - Filtre celles qui ont le label "apex-lock"
    - Compare les titres des issues avec les classes modifiées

 4. GESTION DES CONFLITS
    - Si une classe modifiée correspond à une issue "apex-lock"
    - Le workflow échoue (exit 1)
    - Cela bloque la PR automatiquement
    - Un commentaire explique le problème

# EXEMPLE D'USAGE :
 - Développeur A veut modifier AccountService.cls
 - Il crée une issue : "AccountService - Refactoring en cours" + label "apex-lock"
 - Si développeur B fait une PR qui modifie AccountService.cls
- Le workflow détecte le conflit et bloque la PR
