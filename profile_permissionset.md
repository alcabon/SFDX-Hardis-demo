

# **Rapport sur la Récupération Complète des Métadonnées de Profils et d'Ensembles de Permissions Salesforce DX**

## **Résumé Exécutif**

L'observation de fichiers XML "presque vides" lors de la récupération des profils et des ensembles de permissions Salesforce via les commandes sfdx project retrieve \-m "Profile" et sfdx project retrieve \-m "PermissionSet" est une manifestation courante de la manière dont l'API de métadonnées de Salesforce gère les permissions. Les profils et les ensembles de permissions ne contiennent pas la définition complète des composants auxquels ils accordent l'accès (par exemple, les objets, les champs, les classes Apex) ; ils stockent plutôt les permissions *pour* ces composants. Sans l'inclusion explicite de ces types de métadonnées dépendants dans la requête de récupération, le fichier XML résultant pour les profils et les ensembles de permissions apparaîtra incomplet.

La solution principale à ce défi implique l'utilisation d'un fichier manifeste package.xml pour déclarer explicitement tous les types de métadonnées associés que les profils ou les ensembles de permissions référencent. Ce rapport détaillera comment construire un tel fichier package.xml manuellement, utiliser des outils automatisés pour sa génération, et s'aligner sur l'orientation stratégique de Salesforce vers les ensembles de permissions pour la gestion de l'accès des utilisateurs.

## **1\. La Nuance de la Récupération des Métadonnées Salesforce : Pourquoi les Profils et les Ensembles de Permissions Apparaissent "Vides"**

### **1.1. Comprendre le Comportement de l'API de Métadonnées Salesforce**

L'API de métadonnées de Salesforce est conçue pour gérer les composants de métadonnées de manière modulaire. Les profils et les ensembles de permissions sont des types de métadonnées distincts qui définissent les niveaux d'accès, mais ils n'intègrent pas la définition complète des composants qu'ils sécurisent. Au lieu de cela, ils contiennent des références et des permissions *vers* d'autres types de métadonnées.

Lorsqu'une commande telle que sfdx project retrieve \-m "Profile" ou sfdx project retrieve \-m "PermissionSet" est exécutée sans spécifier d'autres types de métadonnées, l'interface de ligne de commande (CLI) de Salesforce ne récupère que la structure XML de base du profil ou de l'ensemble de permissions. Cette structure liste principalement les *noms* des composants auxquels elle accorde l'accès. Cependant, sans les définitions de composants correspondantes (par exemple, les champs, les objets ou les classes Apex réels), les détails de permission pour ces composants ne peuvent pas être entièrement renseignés dans le fichier XML récupéré. Cela conduit à l'apparence "presque vide" observée par l'utilisateur.1

Le comportement observé, où les fichiers XML des profils et des ensembles de permissions apparaissent incomplets, n'est pas une anomalie mais une conséquence directe de la conception architecturale de l'API de métadonnées de Salesforce. Cette conception sépare les définitions d'accès (dans les profils/ensembles de permissions) des définitions des composants eux-mêmes. Cette approche favorise la modularité et la réduction de la taille des fichiers, mais elle rend impérative une gestion explicite des dépendances lors de la récupération. Le fichier package.xml devient ainsi un élément essentiel pour récupérer des informations complètes sur les profils, car il dicte quels "autres éléments" (dépendances) doivent être associés aux permissions du profil.2

### **1.2. La Nature Relationnelle des Métadonnées de Profil et d'Ensemble de Permissions**

Les profils et les ensembles de permissions fonctionnent comme des couches de contrôle d'accès. Leurs fichiers XML définissent des permissions telles que fieldPermissions, objectPermissions, classAccesses, pageAccesses, recordTypeVisibilities et tabSettings. Ces permissions sont intrinsèquement liées à d'autres composants de métadonnées. Pour que ces permissions soient entièrement récupérées et significatives, les composants référencés eux-mêmes doivent également faire partie de la requête de récupération.

Prenons l'exemple d'un champ personnalisé : un profil ou un ensemble de permissions peut accorder un accès en 'Lecture' ou en 'Modification' à ce champ. La permission elle-même (editable=true, readable=true) réside dans le fichier XML du profil ou de l'ensemble de permissions, mais la définition du champ personnalisé (son nom d'API, son type de données, l'objet auquel il appartient) réside dans le type de métadonnées CustomField. Si CustomField n'est pas récupéré en même temps, l'entrée de permission dans le fichier XML du profil ou de l'ensemble de permissions pourrait être absente ou incomplète, car l'API de métadonnées ne peut pas la contextualiser pleinement.

### **1.3. Exemples Spécifiques de Dépendances Implicites**

Les permissions définies dans les profils et les ensembles de permissions s'étendent à un large éventail de types de métadonnées Salesforce. Pour assurer une récupération complète, ces composants connexes doivent être explicitement inclus dans le fichier package.xml.

Les paramètres d'accès pour les composants gérés suivants peuvent être récupérés et déployés dans les profils et les ensembles de permissions à partir de l'API version 29.0 : classes Apex, applications, permissions de champs personnalisés, permissions d'objets personnalisés, paramètres d'onglets personnalisés, sources de données externes, types d'enregistrements et pages Visualforce.3 Une visibilité de type d'enregistrement de profil, par exemple, nécessite la récupération simultanée du type d'enregistrement et du profil.4

Il existe une perception au sein de la communauté selon laquelle la récupération des ensembles de permissions serait simple, certains suggérant qu'il suffit de récupérer l'ensemble de permissions pour obtenir tout ce à quoi il a accès.4 Cependant, l'expérience de l'utilisateur et d'autres sources faisant autorité indiquent clairement que les ensembles de permissions nécessitent également leurs métadonnées associées pour une récupération complète.1 Cette divergence de vues peut s'expliquer par le fait que la simplicité perçue ne s'applique qu'aux ensembles de permissions très simples, dépourvus de dépendances complexes, ou qu'il s'agit d'une simplification excessive qui ne tient pas dans la plupart des scénarios réels.

Le tableau suivant présente les types de métadonnées clés et leur impact sur la récupération des profils et des ensembles de permissions, fournissant une liste structurée des dépendances les plus courantes et critiques.

**Tableau 1 : Types de Métadonnées Clés et Leur Impact sur la Récupération des Profils/Ensembles de Permissions**

| Type de Métadonnées | Description | Impact sur le XML du Profil/Ensemble de Permissions | Support du Caractère Générique (\*) pour les Membres |
| :---- | :---- | :---- | :---- |
| CustomObject | Objets personnalisés et standard (pour les champs personnalisables) | objectPermissions (lecture, création, modification, suppression, tout afficher, tout modifier) | Oui (pour objets personnalisés) 3 |
| CustomField | Champs personnalisés sur les objets | fieldPermissions (lisible, modifiable) | Non (doit être spécifié explicitement) 6 |
| ApexClass | Classes Apex | classAccesses (accès aux classes) | Oui 7 |
| ApexPage | Pages Visualforce | pageAccesses (accès aux pages) | Oui 7 |
| CustomApplication | Applications personnalisées | applicationVisibilities (visibilité de l'application, par défaut) | Oui 7 |
| RecordType | Types d'enregistrements | recordTypeVisibilities (visibilité du type d'enregistrement, par défaut) | Non (doit être spécifié explicitement) 4 |
| CustomTab | Onglets personnalisés | tabSettings (visibilité de l'onglet) | Oui 7 |
| ExternalDataSource | Sources de données externes | externalDataSourceAccesses (accès à la source de données externe) | Oui 7 |
| Flow | Flux (Flows) | flowAccesses (accès et exécution des flux) | Oui (mais Flow seul ne suffisait pas pour les permissions de flux dans un cas observé) 7 |
| CustomPermission | Permissions personnalisées | customPermissions (attribution de permission personnalisée) | Oui |
| CustomMetadata | Types de métadonnées personnalisés | customMetadataTypeAccesses (accès aux types de métadonnées personnalisés) | Oui 7 |

## **2\. Élaborer des Requêtes de Récupération Complètes : La Puissance de package.xml**

### **2.1. Introduction à package.xml pour la Gestion des Dépendances**

Le fichier package.xml est le manifeste qui définit les composants de métadonnées à récupérer ou à déployer vers une organisation Salesforce à l'aide de la CLI Salesforce (SFDX) ou de l'API de métadonnées. Pour les profils et les ensembles de permissions, il est l'outil indispensable pour garantir que toutes leurs permissions et paramètres associés sont entièrement récupérés.

La commande sfdx project retrieve \-m est un raccourci pour récupérer des types de métadonnées spécifiques. Cependant, pour les scénarios complexes impliquant des dépendances, spécifier un package.xml avec l'option \-x ou \--manifest (par exemple, sfdx force:source:retrieve \-x package.xml ou sf project retrieve start \--manifest package.xml) est l'approche la plus robuste.8

### **2.2. Identification des Types de Métadonnées Essentiels pour une Récupération Complète des Profils et Ensembles de Permissions**

Comme établi, les profils et les ensembles de permissions accordent l'accès à divers composants. Pour récupérer leur configuration complète, ces composants doivent être listés dans le fichier package.xml.

Les types de métadonnées clés à inclure pour une récupération complète des profils et des ensembles de permissions sont : ApexClass, ApexPage, CustomApplication, CustomField, CustomMetadata, CustomObject, CustomTab, ExternalDataSource, Flow, et RecordType.3

### **2.3. Naviguer les Limitations des Caractères Génériques : Quand être Explicite dans package.xml**

Bien que le caractère générique (\*) soit pratique pour récupérer "tous" les composants d'un certain type (par exemple, \<members\>\*\</members\>\<name\>CustomObject\</name\> pour récupérer tous les objets personnalisés), il présente des limitations, en particulier lorsqu'il s'agit des dépendances des profils et des ensembles de permissions.

Pour certains types de métadonnées, spécifier \* ne tirera *pas* automatiquement toutes les permissions associées dans le fichier XML du profil ou de l'ensemble de permissions. Au lieu de cela, il est impératif de lister explicitement chaque membre (par exemple, les noms d'API de champs spécifiques, les noms d'API de types d'enregistrements) si l'on souhaite que leurs permissions apparaissent complètement dans le fichier XML du profil/ensemble de permissions récupéré.6

Une limitation cruciale concerne CustomField et RecordType, où le caractère générique \* ne fonctionne souvent pas pleinement pour renseigner les fichiers XML des profils/ensembles de permissions. Pour garantir que les paramètres granulaires de sécurité au niveau du champ ou de visibilité des types d'enregistrements sont récupérés pour des champs ou des types d'enregistrements spécifiques, il est nécessaire de les lister par leurs noms d'API.6

Cette nuance dans le comportement des caractères génériques pour des types de métadonnées spécifiques signifie qu'une simple stratégie de "tout récupérer" pour les dépendances est souvent insuffisante pour les profils et les ensembles de permissions. Cela contraint les développeurs à maintenir un fichier package.xml très granulaire ou à s'appuyer sur des outils sophistiqués capables d'identifier et de lister dynamiquement ces membres spécifiques. Cette exigence ajoute une couche de complexité à la récupération des métadonnées, en particulier dans les organisations vastes ou en évolution.

Le tableau suivant fournit des exemples concrets de snippets package.xml pour des dépendances courantes, illustrant comment inclure les profils ou les ensembles de permissions aux côtés de leurs dépendances, en montrant l'utilisation des caractères génériques là où c'est approprié et la liste explicite des membres pour des types comme CustomField ou RecordType.

**Tableau 2 : Exemples de Snippets package.xml pour les Dépendances Courantes**

| Scénario | Snippet package.xml | Notes |
| :---- | :---- | :---- |
| **Récupérer le profil 'System Administrator' avec les permissions d'objets et de champs personnalisés** | xml \<?xml version="1.0" encoding="UTF-8"?\> \<Package xmlns="http://soap.sforce.com/2006/04/metadata"\> \<types\> \<members\>Account\</members\> \<members\>MyCustomObject\_\_c\</members\> \<name\>CustomObject\</name\> \</types\> \<types\> \<members\>Account.MyCustomField\_\_c\</members\> \<members\>MyCustomObject\_\_c.AnotherField\_\_c\</members\> \<name\>CustomField\</name\> \</types\> \<types\> \<members\>System Administrator\</members\> \<name\>Profile\</name\> \</types\> \<version\>64.0\</version\> \</Package\> | Les champs personnalisés doivent être listés explicitement (ex: Account.MyCustomField\_\_c). |
| **Récupérer l'ensemble de permissions 'Sales Team Access' avec l'accès aux classes Apex et aux pages Visualforce** | xml \<?xml version="1.0" encoding="UTF-8"?\> \<Package xmlns="http://soap.sforce.com/2006/04/metadata"\> \<types\> \<members\>\*\</members\> \<name\>ApexClass\</name\> \</types\> \<types\> \<members\>\*\</members\> \<name\>ApexPage\</name\> \</types\> \<types\> \<members\>Sales\_Team\_Access\</members\> \<name\>PermissionSet\</name\> \</types\> \<version\>64.0\</version\> \</Package\> | Les classes Apex et les pages Visualforce peuvent généralement utiliser le caractère générique \*.7 |
| **Récupérer l'ensemble de permissions 'Marketing Access' avec la visibilité des types d'enregistrements** | xml \<?xml version="1.0" encoding="UTF-8"?\> \<Package xmlns="http://soap.sforce.com/2006/04/metadata"\> \<types\> \<members\>Lead.Marketing\_Lead\_Record\_Type\</members\> \<members\>Opportunity.Marketing\_Opportunity\_Record\_Type\</members\> \<name\>RecordType\</name\> \</types\> \<types\> \<members\>Marketing\_Access\</members\> \<name\>PermissionSet\</name\> \</types\> \<version\>64.0\</version\> \</Package\> | Les types d'enregistrements doivent être listés explicitement (ex: Lead.Marketing\_Lead\_Record\_Type).6 |
| **Récupérer un profil avec des applications et des onglets personnalisés** | xml \<?xml version="1.0" encoding="UTF-8"?\> \<Package xmlns="http://soap.sforce.com/2006/04/metadata"\> \<types\> \<members\>MyCustomApp\</members\> \<name\>CustomApplication\</name\> \</types\> \<types\> \<members\>MyCustomTab\_\_c\</members\> \<name\>CustomTab\</name\> \</types\> \<types\> \<members\>MyCustomProfile\</members\> \<name\>Profile\</name\> \</types\> \<version\>64.0\</version\> \</Package\> | Les applications et onglets peuvent être listés par leur nom d'API. |

## **3\. Stratégies pour Générer un package.xml Robuste**

### **3.1. Construction Manuelle : Bonnes Pratiques et Pièges Courants**

La construction manuelle d'un fichier package.xml est possible, mais elle peut être complexe et sujette aux erreurs, en particulier dans les grandes organisations avec de nombreux composants personnalisés.8

**Bonnes Pratiques :**

* Commencer avec un modèle (comme ceux disponibles dans les référentiels communautaires).  
* Ajouter progressivement les types de métadonnées et les membres au fur et à mesure des besoins.  
* Utiliser la commande sf project retrieve start \--manifest package.xml pour la récupération.  
* Versionner les fichiers package.xml.

**Pièges Courants :**

* Oublier d'inclure toutes les dépendances nécessaires, ce qui entraîne des fichiers XML de profil/ensemble de permissions incomplets.  
* Mal comprendre les limitations des caractères génériques (par exemple, pour CustomField, RecordType), ce qui se traduit par des permissions granulaires manquantes.  
* La modification manuelle des fichiers XML de profil peut être fastidieuse et entraîner des régressions ou des profils défectueux.8

### **3.2. Tirer Parti des Outils Automatisés et des Extensions VS Code**

Compte tenu de la complexité de package.xml et des dépendances de métadonnées, les outils automatisés simplifient considérablement le processus. La complexité inhérente et la propension aux erreurs de la gestion manuelle des fichiers package.xml pour la récupération complète des profils et des ensembles de permissions 8 font des outils de génération automatisée de

package.xml (tels que les extensions VS Code, les commandes sfdx-hardis ou les constructeurs communautaires) un élément essentiel pour un DevOps Salesforce efficace et fiable. Cette tendance vers l'automatisation est une réponse directe aux défis de la gestion des dépendances de métadonnées.

* **Salesforce Package.xml Generator pour VS Code :** Une extension populaire de VS Code qui offre une interface utilisateur graphique pour sélectionner les composants et générer automatiquement le fichier package.xml.9 Cela simplifie le processus pour les développeurs travaillant dans VS Code.  
* **Commandes SFDX Hardis :** Le plugin sfdx-hardis propose des commandes comme sf hardis:org:generate:packagexmlfull, qui peut générer un package.xml complet pour une organisation entière, y compris les éléments gérés.10 Cela peut être un bon point de départ pour une sauvegarde complète de l'organisation ou pour comprendre toutes les métadonnées d'une organisation.  
* **Autres Outils Communautaires :** Divers autres outils existent, tels que le service Heroku de Ben Edwards (packagebuilder.herokuapp.com), le module npm force-dev-tool, et le JAR Java de Kim Galant.9 Ces outils offrent différentes approches pour générer des fichiers  
  package.xml, des interfaces web aux utilitaires en ligne de commande.

### **3.3. Utilisation de Modèles Fournis par la Communauté pour les Scénarios Courants**

Les référentiels GitHub maintenus par la communauté proposent souvent des modèles package.xml pour divers scénarios de récupération, y compris ceux spécifiquement adaptés aux profils et aux ensembles de permissions avec des dépendances courantes. Ces modèles peuvent servir d'excellents points de départ, réduisant l'effort de construction manuelle.6 Le référentiel

asagarwal/salesforce-package-xml est une ressource précieuse pour de telles structures package.xml préconstruites, conçues pour inclure "tous les autres types de métadonnées qui ont un impact sur le XML du profil" ou "le XML de l'ensemble de permissions".6

Le tableau suivant offre un aperçu des outils de génération de package.xml, en détaillant leurs fonctionnalités, avantages et considérations.

**Tableau 3 : Aperçu des Outils de Génération de package.xml**

| Outil/Méthode | Description | Caractéristiques/Avantages Clés | Considérations |
| :---- | :---- | :---- | :---- |
| **Construction Manuelle** | Création et édition du fichier package.xml à la main. | Contrôle total et granulaire. | Fastidieux, sujet aux erreurs, difficile à maintenir dans les grandes orgs.8 |
| **Salesforce Package.xml Generator pour VS Code** | Extension VS Code qui génère package.xml via une interface graphique. | Interface utilisateur intuitive, sélection visuelle des composants, intégré à VS Code.9 | Nécessite VS Code, peut ne pas gérer toutes les dépendances complexes automatiquement. |
| **sf hardis:org:generate:packagexmlfull** | Commande CLI pour générer un package.xml complet de l'organisation. | Récupération de l'org complète, inclut les éléments gérés, utile pour l'audit ou la sauvegarde initiale.10 | Plugin tiers, peut générer un fichier très volumineux, nécessite un nettoyage manuel pour les déploiements ciblés.10 |
| **force-dev-tool (npm module)** | Outil en ligne de commande pour récupérer et générer package.xml. | Peut récupérer un package.xml complet, scriptable.9 | Nécessite Node.js et npm, courbe d'apprentissage CLI. |
| **JAR Java de Kim Galant** | Outil Java pour inventorier l'org et construire package.xml. | Personnalisable via des fichiers de propriétés, permet d'exclure des éléments par regex.9 | Nécessite Java, configuration via fichiers de propriétés, approche plus technique. |
| **Modèles Communautaires (ex: asagarwal/salesforce-package-xml)** | Fichiers package.xml pré-construits disponibles sur GitHub. | Excellent point de départ, couvre des scénarios courants, réduit l'effort initial.6 | Nécessite des ajustements manuels pour des composants spécifiques (ex: CustomField, RecordType), peut devenir obsolète. |

## **4\. Gestion Moderne des Permissions dans Salesforce : Profils vs. Ensembles de Permissions**

### **4.1. L'Orientation Stratégique de Salesforce : L'Avenir des Permissions**

Salesforce a activement encouragé le passage des profils aux ensembles de permissions pour la gestion des accès utilisateurs. Il s'agit d'une orientation stratégique majeure qui a des implications sur la conception, la récupération et le déploiement des métadonnées.

* **Profils :** Traditionnellement, les profils définissaient les permissions de base d'un utilisateur.11 Cependant, ils sont notoirement difficiles à gérer en raison de leur nature monolithique et de leur taille, contenant souvent des entrées pour chaque champ de chaque objet.5 De plus, les profils "ne sont pas cohérents entre les organisations, et les fichiers sources récupérés et déployés dépendent du type d'organisation, de l'état de suivi et d'autres métadonnées dans l'opération" 2, ce qui rend leur gestion complexe et sujette aux erreurs.  
* **Ensembles de Permissions :** Introduits pour accorder des permissions *additionnelles* et granulaires au-delà du profil d'un utilisateur.11 Ils sont modulaires et flexibles, permettant une attribution et une gestion plus faciles des droits d'accès spécifiques. Salesforce recommande explicitement d'utiliser les ensembles de permissions plutôt que les profils.2

Une implication majeure et future de cette orientation est l'annonce par Salesforce de la **fin de vie (EOL) des permissions sur les profils d'ici le printemps 2026**.5 Cela signifie que les permissions qui résidaient historiquement sur les profils, telles que les permissions d'objets, l'accès aux classes Apex et l'accès aux pages Visualforce, seront transférées aux ensembles de permissions. Bien que les profils ne soient pas complètement retirés, leur rôle dans la gestion des permissions sera considérablement réduit. Ce changement imminent souligne l'urgence d'adopter les ensembles de permissions comme méthode principale de gestion des accès.

Ce changement représente un impératif stratégique pour les organisations Salesforce. Cependant, cette transition n'est pas sans ses propres complexités. Malgré leur modularité, les ensembles de permissions nécessitent toujours une gestion méticuleuse des dépendances lors de la récupération et du déploiement.5 Cela implique la nécessité de mettre à jour les pratiques DevOps et potentiellement d'utiliser des outils plus sophistiqués pour gérer les défis de déploiement d'unité et assurer la cohérence des états de permission entre les environnements.

### **4.2. Bonnes Pratiques pour la Conception et le Déploiement des Ensembles de Permissions**

L'adoption efficace des ensembles de permissions exige le respect des bonnes pratiques pour maintenir un système de gestion des permissions sécurisé, efficace et maintenable.11

* **Principe du Moindre Privilège (PoLP) :** Toujours commencer avec le minimum de permissions et n'accorder que ce qui est nécessaire.11  
* **Noms Descriptifs :** Utiliser des noms et des descriptions clairs et descriptifs pour les ensembles de permissions afin de comprendre facilement leur objectif.11  
* **Conception Modulaire :** Décomposer les fonctionnalités en ensembles de permissions plus petits et ciblés plutôt que de créer des ensembles monolithiques. Cela facilite l'attribution des seules permissions nécessaires.11  
* **Documenter les Dépendances Externes :** Si un ensemble de permissions est lié à un système ou une intégration externe, il est essentiel de le documenter pour éviter toute rupture accidentelle.11  
* **Utiliser la Sécurité au Niveau du Champ (FLS) :** Utiliser la FLS pour un contrôle granulaire sur les champs sensibles.11  
* **Considérer l'Accès API :** S'assurer que les permissions accordées n'exposent pas involontairement des données via des appels API.11  
* **Tester Minutieusement :** Toujours tester les ensembles de permissions nouveaux ou modifiés dans un environnement sandbox avant de les déployer en production.11  
* **Sauvegarder la Configuration :** Sauvegarder régulièrement les configurations des ensembles de permissions.11

### **4.3. Le Rôle des Groupes d'Ensembles de Permissions dans la Rationalisation de la Gestion des Accès**

Les groupes d'ensembles de permissions (PSGs) sont une fonctionnalité puissante introduite par Salesforce pour regrouper plusieurs ensembles de permissions en une seule unité assignable.11 Cela réduit considérablement la complexité de l'attribution de plusieurs ensembles de permissions aux utilisateurs, en particulier dans les organisations avec de nombreux ensembles de permissions. Les PSGs héritent de toutes les permissions de leurs ensembles de permissions contenus, offrant un moyen rationalisé de gérer l'accès pour différents rôles ou fonctions d'utilisateur.

### **4.4. Défis de Déploiement avec les Ensembles de Permissions (Même avec les Bonnes Pratiques)**

Bien que les ensembles de permissions soient la voie recommandée, leur déploiement à l'aide d'outils natifs (tels que les ensembles de modifications ou même SFDX sans une gestion attentive du package.xml) peut toujours présenter des défis similaires à ceux des profils.5

* **Inclusion des Dépendances :** Une frustration majeure est que "tous les composants référencés par l'ensemble de permissions doivent être inclus dans l'ensemble de modifications".5 Si un objet ou un champ personnalisé auquel l'ensemble de permissions accorde l'accès est oublié, les permissions d'objet et de champ seront manquantes dans l'ensemble de permissions déployé.  
* **Déploiement en Unité :** Les ensembles de permissions sont généralement déployés en tant qu'unité complète. Cela signifie que si un ensemble de permissions est déployé à partir d'une organisation source qui contient moins d'objets ou de fonctionnalités que l'organisation cible, l'accès à tous les composants "manquants" dans la source sera désactivé dans l'organisation cible.5 Ce comportement de "déploiement en unité" des ensembles de permissions, où le déploiement à partir d'une organisation source moins complète peut révoquer des permissions dans une organisation cible plus complète, présente un risque significatif de régressions de sécurité involontaires. Cela souligne la nécessité d'outils de déploiement intelligents capables de fusionner les modifications plutôt que de simplement les écraser, et d'une stratégie de déploiement robuste.

## **Conclusion et Recommandations Actionnables**

La récupération complète des profils et des ensembles de permissions dans Salesforce nécessite une compréhension approfondie de la nature relationnelle des métadonnées et une gestion méticuleuse des dépendances. Les fichiers XML de profils et d'ensembles de permissions n'intègrent pas les définitions complètes des composants auxquels ils accordent l'accès ; ils stockent plutôt des références et des définitions d'accès. Par conséquent, une récupération complète exige l'inclusion explicite de tous les types de métadonnées référencés dans un fichier package.xml.

Les caractères génériques (\*) dans package.xml ont des limitations pour certains types de métadonnées, notamment CustomField et RecordType, lorsqu'il s'agit de récupérer des détails de profil/ensemble de permissions. Cela nécessite de lister explicitement les membres pour obtenir une granularité complète. L'automatisation de la génération de package.xml via des outils dédiés et des modèles communautaires est essentielle pour rationaliser ce processus complexe et réduire les erreurs manuelles.

De plus, Salesforce s'oriente stratégiquement vers les ensembles de permissions comme principal moyen de gestion des accès, avec la suppression progressive des permissions sur les profils d'ici le printemps 2026\. Cette transition, bien que bénéfique pour la modularité, introduit de nouvelles complexités en matière de gestion des dépendances et de comportement de déploiement des ensembles de permissions, notamment la nature de "déploiement en unité" qui peut entraîner des révocations de permissions imprévues.

Pour une gestion et une récupération efficaces des métadonnées, les recommandations suivantes sont formulées :

1. **Prioriser les Ensembles de Permissions :** S'aligner sur l'orientation stratégique de Salesforce. Concevoir les nouvelles configurations d'accès à l'aide d'ensembles de permissions et de groupes d'ensembles de permissions. Commencer la migration des permissions existantes basées sur les profils vers les ensembles de permissions dans la mesure du possible, compte tenu de l'échéance du printemps 2026\.  
2. **Maîtriser package.xml pour les Dépendances :** Pour toute récupération de profils ou d'ensembles de permissions, toujours utiliser un fichier manifeste package.xml. Inclure *tous* les types de métadonnées pertinents auxquels le profil/ensemble de permissions accorde l'accès (par exemple, CustomObject, CustomField, ApexClass, RecordType, CustomApplication, Flow, CustomPermission).  
3. **Être Explicite si Nécessaire :** Porter une attention particulière aux types de métadonnées comme CustomField et RecordType. Si des permissions granulaires pour des champs ou des types d'enregistrements spécifiques doivent être entièrement reflétées dans le fichier XML du profil/ensemble de permissions récupéré, lister explicitement leurs noms d'API dans le package.xml plutôt que de se fier uniquement aux caractères génériques.  
4. **Tirer Parti des Outils de Génération Automatisée de package.xml :** Pour réduire l'effort manuel et minimiser les erreurs, intégrer des outils comme l'extension Salesforce Package.xml Generator pour VS Code ou les commandes sfdx-hardis dans le flux de travail de développement. Pour les récupérations complexes ou complètes de l'organisation, ces outils sont indispensables.  
5. **Adopter une Conception Modulaire des Ensembles de Permissions :** Créer des ensembles de permissions plus petits et ciblés 11 et les regrouper à l'aide de groupes d'ensembles de permissions.11 Cette modularité simplifie la gestion et le déploiement.  
6. **Mettre en Œuvre des Stratégies de Déploiement Robustes :** Être conscient du comportement de "déploiement en unité" des ensembles de permissions.5 Envisager d'utiliser des outils de déploiement avancés (au-delà des ensembles de modifications natifs ou des commandes SFDX de base) qui peuvent fusionner intelligemment les modifications des ensembles de permissions, évitant ainsi les révocations de permissions involontaires dans les organisations cibles. Tester minutieusement toutes les modifications de permissions dans un environnement sandbox avant de les déployer en production.  
7. **Documenter et Communiquer :** Maintenir une documentation claire pour les ensembles de permissions, y compris leur objectif et toute dépendance externe.11 Informer les utilisateurs concernés des modifications importantes de permissions.

#### **Sources des citations**

1. Field Permission not showing up in Permission Set Metadata : r ..., consulté le juin 15, 2025, [https://www.reddit.com/r/SalesforceDeveloper/comments/1kjxxa2/field\_permission\_not\_showing\_up\_in\_permission\_set/](https://www.reddit.com/r/SalesforceDeveloper/comments/1kjxxa2/field_permission_not_showing_up_in_permission_set/)  
2. Retrieve Changes to Profiles with Source Tracking | Salesforce DX ..., consulté le juin 15, 2025, [https://developer.salesforce.com/docs/atlas.en-us.sfdx\_dev.meta/sfdx\_dev/sfdx\_dev\_source\_tracking\_source\_tracking\_profiles.htm](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_source_tracking_source_tracking_profiles.htm)  
3. Sample package.xml Manifest Files | Metadata API Developer Guide, consulté le juin 16, 2025, [https://developer.salesforce.com/docs/atlas.en-us.api\_meta.meta/api\_meta/manifest\_samples.htm](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/manifest_samples.htm)  
4. How to Access all Profile Metadata | Salesforce Trailblazer Community \- Trailhead, consulté le juin 15, 2025, [https://trailhead.salesforce.com/trailblazer-community/feed/0D54V00007eryrPSAQ](https://trailhead.salesforce.com/trailblazer-community/feed/0D54V00007eryrPSAQ)  
5. A better way to deploy permission sets in Salesforce \- Gearset, consulté le juin 15, 2025, [https://gearset.com/blog/a-better-way-to-deploy-profiles-and-permission-sets/](https://gearset.com/blog/a-better-way-to-deploy-profiles-and-permission-sets/)  
6. asagarwal/salesforce-package-xml \- GitHub, consulté le juin 15, 2025, [https://github.com/asagarwal/salesforce-package-xml](https://github.com/asagarwal/salesforce-package-xml)  
7. salesforce-package-xml/package-permissionset.xml at main \- GitHub, consulté le juin 15, 2025, [https://github.com/asagarwal/salesforce-package-xml/blob/main/package-permissionset.xml](https://github.com/asagarwal/salesforce-package-xml/blob/main/package-permissionset.xml)  
8. Profile Generator: The Missing Piece for SFDX \- Salesforce Ben, consulté le juin 16, 2025, [https://www.salesforceben.com/profile-generator-the-missing-piece-for-sfdx/](https://www.salesforceben.com/profile-generator-the-missing-piece-for-sfdx/)  
9. ant \- How to generate package.xml automatically for salesforce ..., consulté le juin 15, 2025, [https://stackoverflow.com/questions/29767394/how-to-generate-package-xml-automatically-for-salesforce-source-org](https://stackoverflow.com/questions/29767394/how-to-generate-package-xml-automatically-for-salesforce-source-org)  
10. Initialize sfdx sources from Salesforce org \- Sfdx-Hardis ..., consulté le juin 15, 2025, [https://sfdx-hardis.cloudity.com/salesforce-ci-cd-setup-existing-org/](https://sfdx-hardis.cloudity.com/salesforce-ci-cd-setup-existing-org/)  
11. Salesforce Permission Set Best Practices, consulté le juin 15, 2025, [https://sf.watch/articles/141/salesforce-permission-set-best-practices/](https://sf.watch/articles/141/salesforce-permission-set-best-practices/)  
12. Permission Sets and Profile Settings in Packages | First-Generation ..., consulté le juin 16, 2025, [https://developer.salesforce.com/docs/atlas.en-us.pkg1\_dev.meta/pkg1\_dev/distribution\_perm\_sets\_profile\_settings.htm](https://developer.salesforce.com/docs/atlas.en-us.pkg1_dev.meta/pkg1_dev/distribution_perm_sets_profile_settings.htm)
