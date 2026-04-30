# DEVOPS S8
## Softaware bots in Software Engineering
### SAVATTE Gabriel | ROHOU Loan | ROULLEAU Tom | ROUILLARD Elouan

## Table des matières
- [Introduction](#introduction)
- [Mise en place technique du projet](#mise-en-place-technique-du-projet)
  - [Versions utilisées](#versions-utilisées)
  - [package.json](#changement-dans-le-packagejson)
  - [docker-compose](#modification-du-docker-compose-du-back-)
  - [Lancement du projet](#lancement-du-projet-)
- [Bots et automatismes](#bots-et-automatismes)
  - [Vue générale](#vue-générale-des-bots-découverts)
  - [Dependabot](#dépendabot)
- [Conclusion](#conclusion)
  

## Introduction
Afin de réaliser ce projet nous nous sommes basés sur le [projet Doodle](https://github.com/barais/doodlestudent) mis à disposition. Dans ce document nous détaillerons dans une premiere courte partie ce que nous avons fait pour faire fonctionner le projet puis nous détaillerons dans une seconde partie les différents bots et automatismes que nous avons découverts lors de ce cours. Nous expliciterons également les différentes étapes suivies pour la mise en place de certains de ces bots.

## Mise en place technique du projet

### Versions utilisées
- JAVA : 11
- Maven : 3.8.7
- NodeJS : 17

### Changement dans le `package.json`
Afin d'adapter le projet aux nouvelles versions de NodeJS (la version 17 a été utilisée ici), il etait nécessaire d'ajouter l'option `openssl-legacy-provider` au script `start` situé dans le fichier `package.json`.

### Modification du `docker-compose` du back :

#### Healthcheck de la DB
Ajout d'un healthcheck sur la base de données pour bien attendre que MySQL s'initialise avant que le reste du back ne se lance.

```yml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 5s
  timeout: 5s
  retries: 10
```

#### Ajout du "cerveau" `backend`
On ajoute un service qui va servir à contenir la logique métier de l'application.\

Ses responsabilités sont :
- `build: context: ./` : au lieu de télécharger une image on dit à Docker de trouver le fichier `DockerFile` et de fabriquer l'image lui même. Cela permet d'automatiser le build du code.

- `ports: "8080:8080"` : ouverture d'un tunnel entre la machine et le conteneur. Cela permet au front end d'envoyer des requêtes au serveur Java.

- `depends_on` : On définit l'ordre de priorité. Il ne faut pas que le serveur Java se lance tant que la base de données n'est pas initialisée et fonctionnelle. Cela évite d'éventuels plantages au démarrage.

- `environment` : C'est là qu'on indique au code Java comment trouver les autres services dans le réseau interne de Docker (ex: db:3306 pour la base de données, mail pour le serveur SMTP).

### Lancement du projet :

Pour lancer le projet il suffit d'ouvrir 2 terminaux :
- Un premier pour le back :
```bash
cd api/
docker compose up --build
```
- Un deuxième pour le front :
```bash
cd front/
npm i
npm start
```

## Bots et automatismes

### Vue générale des bots découverts

| Nom du bot | Fonctionnalité(s) du bot | Déployé ? | Source |
| :----------: | ------------------------ | :---------: | :------: |
| Dependabot | - Alerte sur les vulnérabilités des dépendances<br>- PR automatiques lors des MAJ de sécurité | ✅ | [Dependabot](https://github.com/dependabot) |
| RenovateBot | - Maintenance proactive : Automatise la mise à jour des dépendances pour éliminer la dette technique<br>- PR intelligentes : Regroupe et planifie les mises à jour pour éviter de submerger les développeurs | ✅ | [RenovateBot](https://github.com/marketplace/renovate) |
| TruffleHog | - Empêche un build quand des mots de passes ou des données sensibles sont détectées | ✅ | ... |
| GitHub Actions | - Pas vraiment un bot mais automatisation d'actions | ✅ | [Actions](https://github.com/features/actions) |

### Dépendabot
#### Présentation générale
Dependabot est un bot de gestion des dépendances, acquis par GitHub en 2019 et intégré nativement dans les projets hébergés sur GitHub.\
Dans le cycle de vie du développement logiciel, les projets s'appuient sur de nombreuses bibliothèques externes (packages npm, dépendances Maven, images Docker, etc.). Au fil du temps, ces dépendances accumulent de la dette technique à mesure qu'elles vieilissent et, plus grave encore, peuvent présenter des failles de sécurité.\
Le rôle de Dependabot est d'agir comme un veilleur silencieux qui analyse en continu les fichiers de configuration du projet (`package.json`, `pom.xml` etc.) et les compare avec les bases de données de vulnérabilités ainsi qu'avec les dernières version publiées par les mainteneurs de paquets. Son objectif principal est de maintenir le codes des dépendances à jour tout en minimisant l'intervention humaine requise pour efefctuer cette veille technologique.

#### Quelques exemples pour comprendre pourquoi c'est important
Pour illustrer l'interet vital d'un outil automatisé comme Dependabot, il est interessant d'observer la rapidité et l'ampleur que peuvent avoir de récentes attaque de type Supply Chain. Des ces scénarios, les attaquants ne visent pas l'application finale (notre projet par exemple), mais ils infiltrent les bibliothèques que nous utilisons.

##### Le phishing de Josh Junon (Septembre 2025)
En septembre 2025, une campagne de phishing très sophistiquée a permis de voler les identifiants du compte npm de Josh Junon, mainteneur historique de dizaines de bibliothèques JavaScript fondamentales telles que [debug](https://www.npmjs.com/package/debug) ou [chalk](https://www.npmjs.com/package/chalk). En quelques minutes, les attaquants ont publié des versions malveillantes contenant un crypto-clipper (un malware ayant comme objectif de voler les transactions de cryptomonnaies) dans des paquets cumulant plus de 2,6 milliards de téléchargements hebdomadaires et touchant indirectement près de 34 % de l'écosystème npm.

##### La compromission d'Axios (Mars 2026)
Encore plus récemment, à la fin du mois de mars 2026, une attaque attribuée à des acteurs étatiques nord-coréens à frappe le client HTTP [axios](https://www.npmjs.com/package/axios) (qui cumule environ 100 millions de téléchargements par semaine). Les attaquants ont pu compromettre le compte du mainteneur pour publier des versions malveillantes qui incluaient silencieusement une nouvelle dépendance (`plain-crypto.js` [ici pas d'hyperlien pour des raisons évidentes]) servant de cheval de Troie pour infecter les machines des développeurs et les serveurs d'intégration continue.

#### Quel est le rôle de Dépendabot ?
Face à des compromissions d'une telle envergure, le temps de réaction humain est largement insuffisant. Un projet moyen comporte des milliers de dépendances transitives (dépendances de dépendances). Ainsi, dès que la vulnérabilité à été identifiées et qualifiées dans les bases de données, Dependabot a pu alerter instantanément les milliers d'équipes impactées à travers le monde. Il a également généré automatiquement les Pull Request pour rétrograder ou épingler les versions saines, évitant ainsi aux développeurs de devoir auditer manuellement l'integralité de leurs fichiers.

#### Étapes d'installation
1. Activation via l'interface : Dans les paramètres du dépôt GitHub (section "Security and quality" > onglet "Advanced Security"), il suffit d'activer les options "Dependabot alerts" et "Dependabot security updates". C'est cette dernière option qui va se charger d'activer les pull requests automatiques. D'autres options existent comme "Dependabot malware alerts" qui permet d'être notifié en cas de malware (et pas juste pour toute MAJ de sécurité) ou encore "Dependabot version updates" qui permet de génerer des pull requests automatiquement pour garder les dépendances à jour des qu'une nouvelle version est publiée.

2. Configuration fine (pas effectuée dans ce projet) : Pour un contrôle total il est recommandé de créer un fichier `.github/dependabot.yml` à la racine du projet. Ce fichier permet de définir nottament :
   - Le **package-ecosystem** (npm, pip, docker)
   - Le **directory** (le dossier où se trouve le manifeste des dépendances)
   - Le **schedule** (fréquence de vérifications : journalières, hebdomadaires, mensuelles)
   - Des Règles spécifiques, comme l'assignation automatique des PR à certains développeurs ou l'ignorance volontaire de certaines mises à jour mineures.

#### Actions effectuées
- Scan continu et Alertes : Il détecte les dépendances obsolètes ou vulnérables et génère des alertes de sécurité détaillées dans l'onglet Security de GitHub, en précisant le niveau de criticité.

- Création automatique de Pull Requests : Dès qu'une mise à jour est disponible pour corriger une faille ou mettre à jour une version, Dependabot crée une branche isolée et ouvre une PR.

- Documentation embarquée : Chaque PR générée inclut les notes de version , le journal des modifications et les commits de la nouvelle version de la dépendance, permettant aux développeurs de comprendre rapidement ce qui a changé.

#### Regard critique et conclusion

Dependabot est un outil très efficace qui démocratise le concept de DevSecOps. Il réduit drastiquement la surface d'attaque des applications et fait gagner un temps précieux aux équipes.

Cependant, son utilisation présente certaines limites qu'il faut connaître et maîtriser. Le risque principal est la "fatigue des alertes" : si le projet possède beaucoup de dépendances et que Dependabot est configuré pour des mises à jour quotidiennes, les développeurs peuvent être noyés sous les Pull Requests. On a pu le voir dans ce projet : bien que le projet soit "petit" nous avons eu 11 pull requests et 141 alertes de sécurité (dont 17 critiques et 62 hautes). De plus, Dependabot met en lumière une règle du DevOps : l'automatisation des mises à jour n'a de sens que si elle est couplée à une intégration continue robuste. Si le projet manque de tests automatisés, fusionner une PR de Dependabot devient un pari risqué, car une mise à jour (même mineure) peut introduire des changements "breaking changes" qui rendront notre code obsolète. En conclusion, Dependabot est un assistant indispensable, mais il exige en retour une très bonne qualité de code et des tests irréprochables.

### Renovate Bot

#### Présentation générale
Renovate est un outil d'automatisation des dépendances "open-source" et multi-plateformes (GitHub, GitLab, Bitbucket). Contrairement à Dependabot qui est verrouillé sur l'écosystème GitHub, Renovate est devenu le standard de l'industrie pour les projets complexes et multi-langages.

Dans une architecture moderne comme la nôtre (un projet Full Stack avec du Java/Quarkus et de l'Angular), la gestion manuelle des bibliothèques est une tâche titanesque. Renovate agit comme un intendant DevOps : au-delà de la simple veille sécuritaire, il gère proactivement le cycle de vie complet de chaque composant du projet, qu'il s'agisse des librairies de code, des plugins Maven, ou même de la version de Node.js utilisée.

#### Quelques exemples pour comprendre pourquoi c'est important
L'automatisation avec Renovate ne sert pas qu'à corriger des failles, elle sert à éviter la détrition technologique. Rester bloqué sur des versions obsolètes expose le projet à des risques majeurs :
- L'incident "Log4Shell" (Décembre 2021) : Cette vulnérabilité critique dans la bibliothèque de log Java Log4j a forcé des milliers de développeurs à travailler en urgence pendant les fêtes. Les équipes utilisant Renovate ont pu déployer un correctif mondial sur des centaines de dépôts en quelques minutes grâce à une simple règle de configuration, là où d'autres ont dû modifier manuellement chaque fichier pom.xml un par un.
- L'attaque par "Dependency Confusion" : Certains attaquants publient des paquets malveillants sur les registres publics (npm, Maven Central) avec le même nom que des paquets internes à une entreprise. Renovate permet de configurer des sources de confiance et de détecter immédiatement si une dépendance change de provenance, bloquant ainsi une infiltration silencieuse avant même qu'elle n'atteigne les serveurs de production.

#### Quel est le rôle de Renovate Bot ?
Le rôle de Renovate est d'automatiser la maintenance évolutive. Il apporte une dimension stratégique à la maintenance du code :
1. Réduction du "Bruit" : Contrairement aux bots qui ouvrent une PR pour chaque micro-mise à jour, Renovate sait grouper les dépendances. Par exemple, il peut réunir toutes les mises à jour non-critiques du lundi matin dans une seule Pull Request, préservant ainsi la concentration des développeurs.
2. Gestion de la Stabilité : Renovate peut attendre qu'une version ait 3 ou 7 jours d'existence avant de la proposer. Cela évite d'installer une version "buggée" publiée trop rapidement par un mainteneur (une protection contre les attaques de type Supply Chain immédiates).
3. Support Multi-écosystème : Dans notre projet, il analyse simultanément le pom.xml de l'API (Java), le package.json du Front (Angular) et même les fichiers Docker ou les workflows GitHub Actions. Il offre une vision centralisée et cohérente de la santé technologique de toute l'application.

#### Étapes d'installation
Installation via GitHub App : Renovate a été installé comme une application externe autorisée sur le dépôt. Contrairement aux outils natifs, il démarre par une "Onboarding PR" pour valider sa configuration avant d'agir.

Tout le pilotage repose sur le fichier renovate.json à la racine. Nous y avons configuré :
- Le planning (lundi, 9h-17h) pour maîtriser le flux de travail.
- Le regroupement automatique des mises à jour mineures pour éviter de polluer la liste des Pull Requests.
- Le Dependency Dashboard pour garder une vue d'ensemble via l'onglet Issues.

#### Actions effectuées
- Scan multi-projets : Le bot a détecté et analysé simultanément le backend (api/pom.xml) et le frontend (front/package.json).
- Gestion intelligente des versions : Il sépare les montées de versions majeures des petites corrections.
- Il génère des PR détaillées incluant les "Release Notes" pour permettre une revue de code rapide sans quitter GitHub.
- Auto-régulation : Grâce au planning, il prépare les mises à jour en arrière-plan mais ne les propose qu'au créneau prévu, évitant ainsi d'interrompre les développeurs en plein sprint.

#### Regard critique et conclusion
Renovate est un outil de référence pour la maintenance continue (DevOps) car il automatise la mise à jour des dépendances avant même qu'elles ne deviennent un problème. Sa force majeure est la réduction du bruit grâce au groupement des PR, ce qui limite la "fatigue des alertes" constatée avec d'autres outils.
Points de vigilance :
1. Confiance en la CI : Automatiser les mises à jour exige des tests automatisés (Unitaires/E2E) irréprochables ; sans eux, fusionner une PR de bot reste risqué.
2. Maintenance de la config : La puissance de Renovate vient de sa complexité ; une configuration mal maîtrisée peut vite devenir contre-productive.


### TruffleHog
#### Présentation générale
TruffleHog est un outil de sécurité scanner (SAST) spécialisé dans la détection de secrets. Initialement conçu pour fouiller l'historique des commits Git, il est aujourd'hui l'un des outils de référence dans l'écosystème DevSecOps. Il agit comme un filet automatisé pour empêcher la fuite accidentelle de données sensibles.

#### Quel est le rôle de TruffleHog ?
Son rôle est d'analyser le code source et le système de fichiers pour identifier des secrets "hardcodés" (clés API, mots de passe, certificats SSH, tokens cloud). En DevOps, il sert de "Security Gate" dans la pipeline CI/CD : si un secret est détecté, le build échoue et le déploiement est bloqué, protégeant ainsi l'infrastructure de l'entreprise.

#### Étapes d'installation
Dans ce projet, TruffleHog est installé via GitHub Actions en utilisant Docker. Les étapes sont :

   - Création d'un workflow YAML dans `.github/workflows/`.
   - Utilisation de l'action `actions/checkout` pour récupérer le code.
   - Exécution de l'image Docker officielle `trufflesecurity/trufflehog:latest`.
   - Configuration des arguments de scan (ex: `filesystem`, `--fail`, `--exclude-detectors`).

#### Actions effectuées
À chaque "Push" ou "Pull Request", le bot effectue les actions suivantes :

   - Extraction : Il parcourt l'intégralité du répertoire de travail.
   - Analyse : Il compare chaque chaîne de caractères à une base de données de plus de 700 détecteurs (signatures de clés AWS, Stripe, etc.).
   - Vérification (Optionnelle) : Il peut tenter de vérifier si la clé est active.
   - Rapport : Il affiche dans les logs l'emplacement exact (fichier et ligne) des fuites trouvées.
   - Blocage : Il renvoie un code d'erreur (Exit Code 183 ou 1) pour stopper la pipeline si un danger est détecté.

Voic un exemple d'utilisation.
Quand on ajoute intentionnellement des faux mots de passes, observe les logs suivants.

<img width="1010" height="475" alt="trufflehog_scan" src="https://github.com/user-attachments/assets/908a2b7d-4a46-4547-9bca-6c50f5a2ea59" />

Il classe les mots de passe potentiels par **types** : ici il détecte un mot de passe Github et AWS. Quand on supprime les mots de passe du fichier mais que l'on laisse des variables anodines, le build fonctionne. On observe ceci dans les actions.

<img width="485" height="170" alt="builds_trufflehog" src="https://github.com/user-attachments/assets/a733662c-b589-400f-9b8b-1fa730674d7c" />

#### Regard critique et conclusion
TruffleHog est un outil extrêmement puissant mais qui nécessite un ajustement fin (fine-tuning). Il peut générer des "faux positifs" sur des fichiers de configuration internes (.git/config). C'est un pilier du "Shift Left Security" (sécuriser le plus tôt possible). Il ne remplace pas une bonne hygiène de développement, mais il offre une ceinture de sécurité indispensable contre l'erreur humaine.

### Auto-labeler de Pull Requests

#### Présentation générale
Afin d'un peu plus pousser l'automatisation du projet, nous avons décidé de créer notre propre bot.
Dans un projet collaboratif, la gestion des Pull Requests peut rapidement devenir chaotique : manque de labels, mauvaise catégorisation, difficulté à prioriser les revues.
Pour répondre à ce problème, nous avons développé un bot simple basé sur GitHub Actions permettant d’assigner automatiquement des labels aux Pull Requests en fonction des fichiers modifiés.

Ce bot agit comme un assistant organisationnel en classifiant automatiquement les contributions (frontend, backend, documentation).

#### Quel est le rôle ce bot ?
Le bot analyse les fichiers modifiés dans une Pull Request et applique automatiquement des labels prédéfinis :
- Si des fichiers dans `/front/` sont modifiés → label `frontend`
- Si des fichiers dans `/api/` sont modifiés → label `backend`
- Si des fichiers `.md` sont modifiés → label `documentation`

Cela permet :
- Une meilleure lisibilité du projet
- Une priorisation plus rapide des PR
- Une attribution facilitée aux bons développeurs

#### Étapes d'installation
Ce bot est implémenté via un workflow GitHub Actions.

1. Création du fichier :
```
.github/workflows/auto-label.yml
```

2. Configuration du workflow :
```yaml
name: Auto Label PRs

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  pull-requests: write
  issues: write

jobs:
  labeler:
    runs-on: ubuntu-latest
    steps:      
      - name: Label PR based on files
        uses: actions/github-script@v7
        with:
          script: |
            const files = await github.rest.pulls.listFiles({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.issue.number,
            });

            const labels = new Set();

            files.data.forEach(file => {
              if (file.filename.startsWith("front/")) {
                labels.add("frontend");
              }
              if (file.filename.startsWith("api/")) {
                labels.add("api");
              }
              if (file.filename.endsWith(".md")) {
                labels.add("documentation");
              }
            });

            if (labels.size > 0) {
              await github.rest.issues.addLabels({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.issue.number,
                labels: [...labels],
              });
            }

```

(Attention : Il faut s'assurer que les labels existent bel et bien dans le repository au préalable).

#### Actions effectuées
À chaque ouverture ou mise à jour d’une Pull Request, le bot effectue :

- Analyse automatique des chemins de fichiers modifiés.
- Gestion intelligente des PR transverses : grâce à l'utilisation d'un objet Set en JavaScript, si une Pull Request modifie à la fois des fichiers dans /front/ et dans /api/, le bot assignera automatiquement les deux labels (frontend et api) sans créer de doublons.
- Optimisation : notre script interroge directement l'API REST de GitHub. Il n'a pas besoin de télécharger (checkout) tout le code source du projet sur le serveur d'intégration continue. Cela rend l'exécution ultra-rapide (quelques secondes) et très économe en ressources de calcul.

#### Regard critique et conclusion
Ce bot est volontairement simple mais apporte une vraie valeur :

Il permet la réduction du travail manuel, d'avoir une meilleure organisation et une meilleure visibilité sur les contributions. Il facilite également la collaboration en orientant les revues vers les bonnes personnes.

Cependant, il présente certaines limites :
- Il est basé uniquement sur les chemins de fichiers (pas de compréhension métier du code).
- Il nécessite une convention stricte dans l’organisation des dossiers du projet pour être efficace (ex: tous les fichiers liés au client doivent rester dans `front/`).

### Bot 5

### GitHub Actions
#### Présentation
L'automatisation et la validation du code est une étape primordiale dans une démarche DevOps. Pour répondre à ce besoin, nous avons mis en place un workflow GitHub Actions dédié à l'execution automatique des tests sur la partie frontend. Bien que ces Actions ne soient pas des "Bots", ce sont tout de même des actions automatisées, qui se lancement "d'elles mêmes" et qui peuvent produire un résultat (bloquer / accpeter une PR) en fonction d'un résultat. Ces scripts agissent commes des filets de sécurité qui garantissent que les nouvelles modifications n'introduisent pas de régressons avant d'être intégrées au code principal.

#### Déclenchement et traçabilité
Le pipeline est configuré avec la directive `on: [pull_request]`. Cela permet de déclencher le script uniquement lorsqu'un développeur propose d'intégrer son code via une Pull Request. Le workflow agit ainsi comme un point de contrôle bloquant (Gatekeeper) : si les tests échouent, la fusion est bloquée ou fortement déconseillée.

De plus, le workflow utilise les variables de contexte de GitHub (`${{ github.actor }}`, `${{ github.ref }}`, etc.) à de nombreuses reprises pour générer des logs dynamiques. Cela offre une excellente traçabilité : on sait immédiatement qui a lancé l'action, sur quelle branche, et par quel événement. Aussi en cas d'erreur on sait exactement oùle problème est survenu.

#### Dréoulement de l'exécution
Le job s'execute sur un environnement virtuel vierge et éphémère hébergé par GitHub. Les différentes étapes sont :
1. Récupération du code : Utilisation de l'action officielle `actions/checkout@v4` pour cloner le dépôt dans l'environnement virtuel.
2. Installation des dépendances : une fois positionné dans le dossier front, la commande `npm install` télécharge tous les paquets nécessaires au projet (définis dans le package.json).
3. Exécution des tests : le script lance npm test -- --watch=false --browsers=ChromeHeadless.

> [!NOTE]
> Le flag `--watch=false` permet de forcer l'arrêt du processus une fois les tests terminés en empechant le mode `watch` activé par défaut dans les frameworks.\
Le flag `--browsers=ChromeHeadless` permet de simuler un navigateur sans interface graphique afin de simuler le rendu et les interactions du front-end.

#### Conclusion sur le workflow
Cette GitHub Action couvre les besoins fondamentaux de l'intégration Continue. Elle garantit l'intégrité du code et standardise l'environnement de test. Elle agit de manière autonome en se déclenchant quand on en a besoin et sans intervention humaine et en produisant des résultats effectifs (bloquage d'une PR, avertissements etc.).

## Conclusion
