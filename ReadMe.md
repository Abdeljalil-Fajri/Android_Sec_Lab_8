# LAB 8 — Analyse de posture et exposition d'applications mobiles avec BeVigil et Yaazhini

**Cours : Sécurité des applications mobiles**
**Durée estimée : 2 heures**
**Niveau : Débutant**

---

## Vue d'ensemble

Ce laboratoire introduit une méthodologie structurée d'audit défensif d'applications mobiles Android. Deux angles d'analyse complémentaires sont mis en pratique : une collecte de signaux externes via BeVigil, qui interroge les données publiquement accessibles associées à une application, et une inspection du contenu interne de l'APK via Yaazhini, qui décompose le paquet pour en révéler la structure, les configurations et les éventuels éléments sensibles embarqués.

L'enchaînement des tâches suit un processus d'audit réel, de la définition du périmètre jusqu'à la production d'un rapport structuré, en passant par la normalisation des résultats et leur mise en correspondance avec les standards de l'industrie.

---

## Objectifs pédagogiques

À l'issue de ce laboratoire, l'apprenant sera en mesure de :

- Mettre en place un environnement de travail tracé et organisé, conforme aux exigences d'un audit professionnel.
- Utiliser BeVigil pour collecter des signaux d'exposition externe : endpoints, domaines, emails, technologies détectées.
- Utiliser Yaazhini pour analyser statiquement le contenu d'un APK et identifier des indices de mauvaise configuration ou de fuite d'information.
- Consolider les résultats issus de deux sources distinctes, éliminer les doublons et évaluer la pertinence de chaque constat.
- Relier les observations aux catégories du standard OWASP MASVS pour contextualiser les risques.
- Rédiger un rapport d'analyse synthétique, exploitable par un décideur comme par un développeur.

---

## Prérequis

- Connaissances de base en sécurité informatique.
- Poste Windows avec accès à Internet et à PowerShell.
- Accès aux outils BeVigil et Yaazhini, fournis par l'enseignant.
- APK pédagogique ou domaine explicitement autorisé mis à disposition pour l'analyse.

---

## Règles et périmètre (à lire avant toute action)

Ce laboratoire s'inscrit dans un cadre strictement légal et défensif. Toute action doit rester dans les limites suivantes :

Sont autorisés : l'analyse de l'APK pédagogique fourni par l'enseignant, d'une application interne expressément désignée, ou d'un domaine dont l'autorisation est documentée.

Sont formellement interdits : toute tentative d'exploitation des failles identifiées, tout test intrusif, tout contournement de mécanismes de sécurité, et toute analyse portant sur une cible non explicitement autorisée.

Toute donnée sensible découverte au cours de l'analyse doit être masquée dans les livrables. Aucune action susceptible de perturber, de surcharger ou d'endommager un système cible n'est tolérée.

---

## Glossaire

**APK** : Android Package Kit, format d'empaquetage des applications Android distribuées sur le Play Store ou en sideloading.

**Asset** : ressource embarquée ou référencée par une application (fichier de configuration, image, certificat, etc.).

**BeVigil** : plateforme d'intelligence sur la sécurité mobile développée par CloudSEK, permettant d'identifier les données publiquement associées à une application.

**Endpoint** : point d'accès à un service ou une API web, généralement sous la forme d'une URL ciblant une fonction précise.

**Exposition** : visibilité non intentionnelle d'une information ou d'une fonctionnalité, constituant un risque potentiel même en l'absence d'exploitation active.

**Faux positif** : alerte générée par un outil qui, après vérification, ne correspond pas à une vulnérabilité réelle dans le contexte analysé.

**Hardcoding** : pratique consistant à inscrire directement dans le code source des valeurs qui devraient rester configurables ou secrètes, comme des clés API ou des mots de passe.

**MASVS** : Mobile Application Security Verification Standard, référentiel OWASP définissant les exigences de sécurité pour les applications mobiles.

**MASTG** : Mobile Application Security Testing Guide, guide méthodologique OWASP décrivant les techniques de test de sécurité mobile.

**OSINT** : Open Source Intelligence, discipline consistant à collecter et analyser des informations provenant de sources librement accessibles.

**Posture de sécurité** : évaluation globale du niveau de protection d'un système, tenant compte à la fois des mesures en place et des expositions résiduelles.

**Secret** : donnée confidentielle (clé API, jeton d'authentification, mot de passe) qui ne doit jamais apparaître en clair dans le code ou les journaux.

**Triage** : processus de classification, de déduplication et de priorisation des constats issus d'une analyse, en vue de leur traitement ordonné.

**Yaazhini** : outil d'analyse statique d'applications mobiles, capable de décompiler un APK et d'en extraire des informations sur le code, les permissions, les configurations et les données embarquées.

---

## Workflow général

```
Préparation & Périmètre  →  Analyse BeVigil  →  Analyse Yaazhini  →  Triage
                                                                          ↓
Nettoyage & Clôture  ←  Rédaction Rapport  ←  Corrélation OWASP  ←  Normalisation
```

---

## Task 0 — Définition du périmètre et cadre éthique (5 min)

**Enjeu :** toute activité d'analyse de sécurité, même dans un contexte pédagogique, doit reposer sur un périmètre écrit et vérifiable. Ce document protège l'analyste et justifie la légitimité de chaque action entreprise.

Créer le dossier `00-scope/` et y placer un fichier `scope.md` contenant les informations suivantes : nom de la cible, propriétaire, référence à l'autorisation (nom de l'enseignant, intitulé du cours), type d'artefact utilisé, liste des actions interdites, date de début et durée prévue.

```powershell
mkdir 00-scope

@"
# Périmètre d'analyse

## Cible autorisée
Nom: [Nom de l'application ou du domaine]
Propriétaire: [Propriétaire de l'application]

## Autorisation
Source: [Référence au cours / intitulé du TP]
Preuve: [Nom de l'enseignant / référence du document d'autorisation]

## Type d'artefact
- [x] APK pédagogique fourni par l'enseignant
- [ ] Application interne autorisée
- [ ] Domaine explicitement autorisé

## Limites de l'analyse
- Observation uniquement, pas d'exploitation
- Pas de tests intrusifs ni de fuzzing
- Pas de contournement de mécanismes de sécurité
- Périmètre strictement limité à la cible désignée

## Période d'analyse
Date de début: $(Get-Date -Format "yyyy-MM-dd")
Durée prévue: 2 heures
"@ | Out-File -FilePath "00-scope\scope.md" -Encoding utf8
```

**Validation :** le fichier `00-scope/scope.md` doit exister, être complet, et toutes ses informations doivent être vérifiables.

**Erreur fréquente :** un périmètre formulé de manière vague ("analyser la sécurité de l'application") sans mention des limites ni de la source d'autorisation n'a aucune valeur probante.

---

## Task 1 — Préparation du workspace et traçabilité (10 min)

**Enjeu :** un audit reproductible requiert une structure de travail cohérente et un journal complet des actions effectuées. Ces éléments permettent à une tierce personne de vérifier la méthodologie et de reproduire les résultats.

Mettre en place l'arborescence suivante et initialiser les fichiers de suivi :

```
lab-mobile-security/
├── 00-scope/
├── 01-bevigil/
├── 02-yaazhini/
├── 03-triage/
└── 04-report/
```

```powershell
mkdir 01-bevigil
mkdir 02-yaazhini
mkdir 03-triage
mkdir 04-report

@"
Date: $(Get-Date -Format "yyyy-MM-dd")
Analyste: [Prénom Nom]
Cible: [Nom de l'application ou du domaine]
Artefact: [Nom du fichier APK ou liste des domaines]
Provenance: [Enseignant / Projet de cours]
Hash: [À compléter si APK]
Versions outils:
  - BeVigil: [version]
  - Yaazhini: [version]
Environnement: Windows $(Get-ComputerInfo | Select-Object -ExpandProperty WindowsProductName)
"@ | Out-File -FilePath "analyse_info.txt" -Encoding utf8

"# Journal des commandes — $(Get-Date -Format "yyyy-MM-dd HH:mm:ss")" | Out-File -FilePath "commands.log" -Encoding utf8
"mkdir 00-scope, 01-bevigil, 02-yaazhini, 03-triage, 04-report" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

**Validation :** l'arborescence est conforme, `analyse_info.txt` contient tous les champs, `commands.log` est initialisé.

**Point de méthode :** toutes les commandes exécutées par la suite doivent être ajoutées au fichier `commands.log` via `-Append`. Cette discipline, contraignante au début, devient un réflexe indispensable dans un contexte professionnel.

---

## Task 2 — Préparation de l'artefact autorisé (10 min)

**Enjeu :** identifier avec précision l'objet de l'analyse et en calculer l'empreinte numérique garantit l'intégrité de la chaîne de preuve.

### Option A — APK fourni par l'enseignant

```powershell
Copy-Item -Path "[chemin_vers_apk]\application_pedagogique.apk" -Destination "00-scope\"

$hash = Get-FileHash -Path "00-scope\application_pedagogique.apk" -Algorithm SHA256
$hash.Hash

(Get-Content -Path "analyse_info.txt") -replace "Hash: \[À compléter si APK\]", "Hash: $($hash.Hash)" | Set-Content -Path "analyse_info.txt"

"Copy-Item -Path '[chemin]' -Destination '00-scope\'" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
"Get-FileHash -Path '00-scope\application_pedagogique.apk' -Algorithm SHA256" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

### Option B — Domaine autorisé

```powershell
@"
example.com
api.example.com
mobile.example.com
"@ | Out-File -FilePath "00-scope\targets.txt" -Encoding utf8

"Created targets.txt with authorized domains" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

**Validation :** l'APK est présent dans `00-scope/` avec son hash documenté, ou `targets.txt` liste les domaines autorisés.

**Point de méthode :** le hash SHA-256 est l'empreinte digitale du fichier. Si l'APK est modifié, même d'un seul octet, le hash change. C'est ce qui permet de certifier que l'artefact analysé est exactement celui qui a été fourni.

---

## Task 3 — Prise en main de BeVigil (15 min)

**Enjeu :** BeVigil collecte des signaux publiquement disponibles sur une application mobile — URLs exposées, endpoints d'API référencés, technologies utilisées, adresses email présentes dans le code. Ces informations permettent d'esquisser la surface d'attaque externe sans interagir directement avec les systèmes cibles.

**Procédure :**

1. Accéder à l'interface BeVigil selon les instructions de l'enseignant (web ou CLI).
2. Créer un projet nommé `LAB-BEGINNER-YYYYMMDD` en remplaçant la date par celle du jour.
3. Importer la cible autorisée (par nom d'application, identifiant Play Store, ou domaine).
4. Parcourir les sections de l'interface : Dashboard, Assets, Endpoints, URLs, Technologies.
5. Exporter les résultats au format JSON.
6. Déplacer l'export dans `01-bevigil/bevigil_export.json`.
7. Documenter la version de BeVigil dans `analyse_info.txt`.

```powershell
Move-Item -Path "[chemin_téléchargement]\[nom_fichier].json" -Destination "01-bevigil\bevigil_export.json"

(Get-Content -Path "analyse_info.txt") -replace "- BeVigil: \[version\]", "- BeVigil: [version_réelle]" | Set-Content -Path "analyse_info.txt"

"Exported BeVigil results to 01-bevigil\bevigil_export.json" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

**Ce qu'il faut observer :** la présence ou l'absence de résultats est déjà informative. Une application bien configurée expose peu de signaux externes. Une application mal conçue peut révéler des endpoints d'API internes, des adresses email de développeurs, ou des bibliothèques tierces obsolètes.

**Point de vigilance :** BeVigil fournit des indices, pas des preuves. Un endpoint détecté n'est pas nécessairement vulnérable. Chaque signal devra être contextualisé dans les étapes suivantes.

**Erreurs fréquentes :** analyser une cible hors périmètre par erreur de saisie ; confondre la recherche par nom d'application et la recherche par domaine ; oublier d'exporter les résultats avant de fermer la session.

---

## Task 4 — Collecte BeVigil : cartographie de la surface d'attaque externe (20 min)

**Enjeu :** une fois les résultats obtenus, l'analyste doit les parcourir méthodiquement et documenter ce qu'il observe dans cinq catégories distinctes, en distinguant rigoureusement les faits avérés des hypothèses.

Créer le fichier de notes structuré :

```powershell
@"
# Notes d'analyse BeVigil

## Ce qui est certain
- [Éléments directement visibles dans le rapport, avec localisation précise]

## Ce qui est hypothèse
- [Éléments supposés, avec justification de l'inférence]

## Points d'intérêt
- [Signaux méritant une investigation complémentaire]

## Domaines et sous-domaines identifiés
-

## Endpoints et chemins d'API
-

## URLs HTTP non chiffrées
-

## Adresses email et identifiants
-

## Technologies et versions détectées
-
"@ | Out-File -FilePath "01-bevigil\bevigil_notes.md" -Encoding utf8

"Created bevigil_notes.md" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

**Méthode d'analyse :**

Pour chaque section du rapport BeVigil, noter tous les éléments présents, leur localisation dans le rapport, et pourquoi ils sont potentiellement significatifs d'un point de vue sécurité. Accorder une attention particulière aux URLs en HTTP (non chiffrées), aux technologies dont la version est connue pour contenir des vulnérabilités, et aux adresses email qui pourraient faciliter des attaques de social engineering.

**Validation :** au moins cinq éléments documentés, chacun associé à sa localisation précise dans le rapport et à une hypothèse d'impact.

**Point de méthode :** la séparation entre "certain" et "hypothèse" n'est pas une formalité. Un rapport qui présente des suppositions comme des faits perd toute crédibilité. Cette discipline est au cœur du travail d'analyste.

---

## Task 5 — Prise en main de Yaazhini (15 min)

**Enjeu :** là où BeVigil observe l'application de l'extérieur, Yaazhini en décompose le contenu. L'outil décompile l'APK, inspecte le manifeste Android, analyse les permissions déclarées, parcourt le code à la recherche de patterns à risque, et génère un rapport structuré couvrant les composants exposés, les communications réseau, le stockage de données et d'autres vecteurs d'exposition interne.

### Exécutable Windows

```powershell
mkdir temp_analysis

& "[chemin_yaazhini]\yaazhini.exe" -apk "00-scope\application_pedagogique.apk" -output "temp_analysis"

Copy-Item -Path "temp_analysis\*" -Destination "02-yaazhini\" -Recurse
Remove-Item -Path "temp_analysis" -Recurse -Force

(Get-Content -Path "analyse_info.txt") -replace "- Yaazhini: \[version\]", "- Yaazhini: [version_réelle]" | Set-Content -Path "analyse_info.txt"

"Ran Yaazhini on APK, results in 02-yaazhini/" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

### Script Python

```powershell
python "[chemin_yaazhini]\yaazhini.py" --apk "00-scope\application_pedagogique.apk" --output "02-yaazhini"

"python yaazhini.py --apk 00-scope\application_pedagogique.apk --output 02-yaazhini" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

**Ce qu'il faut observer :** la structure du rapport généré avant d'en analyser le contenu. Comprendre ce que chaque section couvre permet d'orienter efficacement la lecture lors de la tâche suivante.

**Point de vigilance :** Yaazhini peut produire un volume conséquent de données. Ne pas plonger immédiatement dans les détails ; commencer par identifier les sections à risque élevé.

**Validation :** rapport présent dans `02-yaazhini/`, version documentée dans `analyse_info.txt`.

---

## Task 6 — Collecte Yaazhini : indices d'exposition interne (20 min)

**Enjeu :** le rapport Yaazhini contient des informations que BeVigil ne peut pas collecter, car elles résident à l'intérieur de l'application. L'objectif est d'identifier des éléments qui révèlent une mauvaise pratique de développement ou une exposition involontaire de données.

Créer le fichier de notes structuré :

```powershell
@"
# Notes d'analyse Yaazhini

## Élément 1 : [Type / Nom]
- Localisation : [Section du rapport / chemin dans l'APK]
- Description : [Ce qui a été observé]
- Impact potentiel : [Conséquence si exploité]
- Remédiation suggérée : [Action corrective recommandée]

## Élément 2 : [Type / Nom]
- Localisation :
- Description :
- Impact potentiel :
- Remédiation suggérée :

[Répéter pour au moins 5 éléments]
"@ | Out-File -FilePath "02-yaazhini\yaazhini_notes.md" -Encoding utf8

"Created yaazhini_notes.md" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

**Catégories à inspecter en priorité :**

Secrets potentiels : rechercher dans le rapport les sections "Secrets", "API Keys", "Credentials". Pour chaque secret identifié, ne jamais copier la valeur réelle — documenter uniquement le type et la localisation.

Endpoints hardcodés : repérer les URLs complètes inscrites directement dans le code. Distinguer les endpoints de production des URLs de développement ou de test.

Configurations à risque : vérifier dans le manifeste Android la présence de `android:debuggable="true"`, `android:allowBackup="true"`, `android:usesCleartextTraffic="true"`. Chacun de ces flags constitue un affaiblissement de la posture de sécurité.

Permissions excessives : recenser les permissions sensibles déclarées (`CAMERA`, `ACCESS_FINE_LOCATION`, `READ_CONTACTS`, etc.) et évaluer si elles sont justifiées par les fonctionnalités annoncées de l'application.

Composants exposés : identifier les activités, services ou broadcast receivers déclarés avec `exported="true"` sans protection par permission.

**Validation :** au moins cinq éléments documentés avec les quatre champs requis, aucun secret affiché en clair.

---

## Task 7 — Normalisation et dédoublonnage (15 min)

**Enjeu :** BeVigil et Yaazhini peuvent identifier les mêmes problèmes sous des formes différentes. Consolider les résultats dans un fichier unique évite la redondance, améliore la lisibilité et prépare une base solide pour le rapport.

```powershell
"ID,Source,Élément,Preuve,Confiance,Sévérité,Impact,Recommandation,Référence OWASP,Statut" | Out-File -FilePath "03-triage\triage.csv" -Encoding utf8

"Created triage.csv" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

**Procédure de remplissage :**

Parcourir `bevigil_notes.md` et `yaazhini_notes.md`. Pour chaque élément, attribuer un identifiant unique (`FIND-001`, `FIND-002`, etc.), renseigner la source (`BeVigil`, `Yaazhini`, ou `BeVigil+Yaazhini` si le même constat apparaît dans les deux), et évaluer la confiance (Faible / Moyenne / Forte) en fonction de la qualité de la preuve disponible.

La sévérité s'évalue selon l'impact potentiel : `High` pour un accès non autorisé à des données sensibles, `Medium` pour un affaiblissement de la posture sans exploitation immédiate évidente, `Low` pour un risque conditionnel ou marginal, `Info` pour une observation sans impact direct.

Si un même constat apparaît dans les deux outils, le consolider en une seule ligne avec la source `BeVigil+Yaazhini` et une confiance renforcée.

Atteindre au minimum dix lignes. Si les analyses ont produit moins de dix constats distincts, documenter les aspects qui n'ont révélé aucun problème sous forme d'entrées "RAS" (Rien À Signaler) avec justification.

**Validation :** `triage.csv` présent avec toutes les colonnes, au moins dix lignes, doublons fusionnés.

---

## Task 8 — Corrélation avec OWASP MASVS (15 min)

**Enjeu :** relier chaque constat à une catégorie du Mobile Application Security Verification Standard donne une dimension universelle aux observations. Cela permet de les communiquer dans un langage compris par les équipes de développement, les responsables sécurité et les auditeurs externes.

Les catégories principales du MASVS sont les suivantes :

- `MASVS-STORAGE` : protection des données au repos
- `MASVS-CRYPTO` : usage correct de la cryptographie
- `MASVS-AUTH` : authentification et gestion des sessions
- `MASVS-NETWORK` : sécurité des communications réseau
- `MASVS-PLATFORM` : interaction avec la plateforme Android
- `MASVS-CODE` : qualité et robustesse du code
- `MASVS-RESILIENCE` : résistance à l'analyse et à la manipulation

```powershell
@"
# Correspondances OWASP MASVS

## FIND-001 : [Titre du constat]
- Catégorie MASVS : [MASVS-XXX]
- Référence : [VX.X]
- Justification : [En une phrase, expliquer pourquoi ce constat relève de cette catégorie]

## FIND-002 : [Titre du constat]
- Catégorie MASVS :
- Référence :
- Justification :

[Répéter pour au moins 5 constats]
"@ | Out-File -FilePath "03-triage\owasp_mapping.md" -Encoding utf8

"Created owasp_mapping.md" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

Mettre à jour la colonne "Référence OWASP" dans `triage.csv` pour chacun des constats mappés.

**Point de vigilance :** le mapping doit être motivé par la nature réelle du problème, pas par une correspondance superficielle. Un secret hardcodé relève de `MASVS-STORAGE`, pas de `MASVS-CODE`, parce que l'enjeu est la confidentialité de la donnée, pas la qualité du code en elle-même.

**Validation :** au moins cinq mappings justifiés dans `owasp_mapping.md`, colonne OWASP renseignée dans `triage.csv`.

---

## Task 9 — Rédaction du rapport final (20 min)

**Enjeu :** un rapport d'analyse n'est utile que s'il peut être lu et exploité par quelqu'un qui n'a pas participé à l'analyse. Il doit être suffisamment technique pour justifier les constats, et suffisamment clair pour que les actions correctives soient comprises sans ambiguïté.

```powershell
@"
# Rapport d'analyse de sécurité mobile

## A. Informations générales
- Date : $(Get-Date -Format "yyyy-MM-dd")
- Analyste : [Prénom Nom]
- Cible : [Nom de l'application ou du domaine]
- Version / Hash : [Version ou empreinte SHA-256]
- Outils utilisés : BeVigil v[X.Y.Z], Yaazhini v[X.Y.Z]

## B. Résumé exécutif
[5 lignes maximum. Indiquer le nombre total de constats, leur répartition par sévérité,
le niveau de risque global, et les deux ou trois catégories de problèmes les plus représentées.]

## C. Top 5 constats

### 1. [Titre — FIND-00X]
- Sévérité : [High / Medium / Low / Info]
- Preuve : [Localisation précise dans les artefacts d'analyse]
- Impact : [Conséquence concrète si le problème n'est pas corrigé]
- Remédiation : [Action corrective spécifique et actionnable]
- Référence OWASP : [MASVS-XXX]

[Répéter pour les constats 2 à 5, par ordre décroissant de sévérité]

## D. Faux positifs notables
[Pour chaque faux positif significatif : description de l'alerte et explication de la raison
pour laquelle elle ne constitue pas une vulnérabilité dans ce contexte.]

## E. Recommandations prioritaires
1. [Action 1 — verbe + objet + précision technique]
2. [Action 2]
3. [Action 3]

## F. Annexes
- Export BeVigil : ../01-bevigil/bevigil_export.json
- Rapport Yaazhini : ../02-yaazhini/[nom_du_rapport]
- Triage complet : ../03-triage/triage.csv
- Mapping OWASP : ../03-triage/owasp_mapping.md
"@ | Out-File -FilePath "04-report\rapport_final.md" -Encoding utf8

"Created rapport_final.md" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

**Point de méthode :** un rapport rigoureux distingue trois registres clairement séparés — ce qui a été observé (fait), ce qui est inféré (hypothèse), et ce qui est recommandé (action). Mélanger ces trois registres produit un document ambigu qui peut induire en erreur le lecteur.

**Validation :** toutes les sections sont présentes et complètes, les cinq constats sont documentés avec tous leurs champs, aucune donnée sensible n'est exposée en clair.

---

## Task 10 — Nettoyage et clôture (5 min)

**Enjeu :** un workspace d'analyse qui n'est pas nettoyé après usage présente deux risques : contaminer une session ultérieure avec des données résiduelles, et exposer des informations sensibles à quiconque accèderait au poste.

```powershell
@"
# Checklist de clôture

## Périmètre et traçabilité
- [ ] Scope clairement défini et respecté tout au long de l'analyse
- [ ] Fichier analyse_info.txt complété
- [ ] Journal commands.log à jour

## Collecte et analyse
- [ ] Export BeVigil sauvegardé dans 01-bevigil/
- [ ] Rapport Yaazhini sauvegardé dans 02-yaazhini/
- [ ] Notes d'analyse complètes pour les deux outils

## Triage et reporting
- [ ] triage.csv rempli avec au moins 10 constats
- [ ] Mapping OWASP réalisé pour au moins 5 constats
- [ ] Rapport final complet et structuré dans 04-report/

## Sécurité des livrables
- [ ] Aucun secret affiché en clair dans les fichiers produits
- [ ] Aucune donnée personnelle exposée
- [ ] Aucune technique d'exploitation documentée

Je soussigné(e) [Prénom Nom] certifie avoir conduit cette analyse dans le strict respect
du périmètre autorisé et des règles éthiques définies en début de séance.

Date : $(Get-Date -Format "yyyy-MM-dd")
Signature : [Prénom Nom]
"@ | Out-File -FilePath "checklist_fin.md" -Encoding utf8

"Created and completed checklist_fin.md" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

Vérifier l'ensemble des fichiers produits à la recherche de mots-clés sensibles (`password`, `key`, `token`, `secret`) avant de soumettre ou d'archiver le workspace.

---

## Troubleshooting

**BeVigil ne retourne aucun résultat**
Vérifier l'orthographe exacte du nom de l'application ou de l'identifiant Play Store. Essayer une variante ou une recherche par domaine associé. Si l'application n'est pas indexée, documenter ce fait dans `bevigil_notes.md` et concentrer l'analyse sur Yaazhini.

```powershell
"ISSUE: BeVigil — aucun résultat pour [cible]" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

**Yaazhini échoue à analyser l'APK**
Vérifier que le fichier APK n'est pas corrompu en recalculant son hash et en le comparant à la valeur documentée. Vérifier les droits d'accès au fichier et au dossier de sortie. Si l'erreur persiste, redémarrer l'outil et documenter le message d'erreur exact.

```powershell
Test-Path "00-scope\application_pedagogique.apk"
Get-FileHash -Path "00-scope\application_pedagogique.apk" -Algorithm SHA256
"ISSUE: Yaazhini — erreur lors du parsing de l'APK" | Out-File -FilePath "commands.log" -Encoding utf8 -Append
```

**Volume de faux positifs trop important**
Appliquer systématiquement le critère triple : preuve directe + impact concret + reproductibilité. Un constat qui ne satisfait pas ces trois critères est un faux positif jusqu'à preuve du contraire. Les documenter dans une section dédiée plutôt que de les ignorer.

```powershell
"# Faux positifs identifiés et justifiés" | Out-File -FilePath "03-triage\faux_positifs.md" -Encoding utf8
```

**Export des résultats impossible**
Capturer des captures d'écran des sections pertinentes. Documenter manuellement les constats observés avec référence à la version de l'outil et à la date de l'analyse.

```powershell
"# Résultats documentés manuellement (export indisponible)" | Out-File -FilePath "01-bevigil\manual_results.md" -Encoding utf8
```

**Difficulté à évaluer la sévérité d'un constat**
Utiliser la grille suivante comme point de départ, puis ajuster au contexte de l'application :

- `High` : compromission directe de la confidentialité, de l'intégrité ou de la disponibilité de données sensibles.
- `Medium` : affaiblissement de la posture de sécurité sans exploitation triviale.
- `Low` : risque conditionnel, nécessitant des circonstances particulières.
- `Info` : observation sans impact sécurité direct mais méritant d'être documentée.

En cas de doute persistant, classer `Medium` et justifier explicitement l'incertitude dans le rapport.

---

## Checklist de début

- [ ] Accès aux outils BeVigil et Yaazhini confirmé.
- [ ] APK pédagogique ou domaine autorisé disponible et identifié.
- [ ] Règles et périmètre lus et compris.
- [ ] Structure de dossiers créée.
- [ ] Fichiers de traçabilité initialisés.

---

## Checklist de fin

- [ ] Scope défini et respecté.
- [ ] `analyse_info.txt` complété, hash APK documenté.
- [ ] Export BeVigil sauvegardé dans `01-bevigil/`.
- [ ] Rapport Yaazhini sauvegardé dans `02-yaazhini/`.
- [ ] Notes d'analyse rédigées pour les deux outils.
- [ ] `triage.csv` rempli avec au moins dix constats.
- [ ] Mapping OWASP réalisé pour au moins cinq constats.
- [ ] `rapport_final.md` complet et structuré.
- [ ] Aucun secret en clair dans les livrables.
- [ ] `checklist_fin.md` signée.

---

## Livrables attendus

```
lab-mobile-security/
├── analyse_info.txt
├── commands.log
├── checklist_fin.md
├── 00-scope/
│   ├── scope.md
│   └── targets.txt              (si option domaine)
│   └── application_pedagogique.apk  (si option APK)
├── 01-bevigil/
│   ├── bevigil_export.json
│   └── bevigil_notes.md
├── 02-yaazhini/
│   ├── [rapport Yaazhini]
│   └── yaazhini_notes.md
├── 03-triage/
│   ├── triage.csv
│   └── owasp_mapping.md
└── 04-report/
    └── rapport_final.md
```

---

*Laboratoire développé dans le cadre du cours Sécurité des applications mobiles.*
