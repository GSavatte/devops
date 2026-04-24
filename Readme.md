# DEVOPS S8
## Softaware bots in Software Engineering
### SAVATTE Gabriel | ROHOU Loan | ROULLEAU Tom | ROUILLARD Elouan

## Introduction
Afin de réaliser ce projet nous nous sommes basés sur le [projet Doodle](https://github.com/barais/doodlestudent) mis à disposition. Dans ce document nous détaillerons dansune premiere courte partie ce que nous avons fait pour faire fonctionner le projet puis nous détaillerons dans une seconde partie les différents bots et automatismes que nous avons découverts lors de ce cours. Nous expliciterons également les différentes étapes suivies pour la mise en place de certains de ces bots.

## Mise en place technique du projet

### Changement dans le `package.json`
Afin d'adapter le projet aux nouvelles versions de NodeJS (la version 17 a été utilisée ici), il etait nécessaire d'ajouter l'option `openssl-legacy-provider` au script `start` situé dans le fichier `package.json`.

## Bots et automatismes

### Vue générale des bots découverts

| Nom du bot | Fonctionnalité(s) du bot | Déployé ? | Source |
| :----------: | ------------------------ | :---------: | :------: |
| Dependabot | - Alerte sur les vulnérabilités des dépendances<br>- PR automatiques lors des MAJ de sécurité | ✅ | [Dependabot](https://github.com/dependabot) |

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

## Conclusion