# DEVOPS S8
## Softaware bots in Software Engineering
### SAVATTE Gabriel | ROHOU Loan | ROULLEAU Tom | ROUILLARD Elouan

## Introduction
Afin de réaliser ce projet nous nous sommes basés sur le [projet Doodle](https://github.com/barais/doodlestudent) mis à disposition. Dans ce document nous détaillerons dansune premiere courte partie ce que nous avons fait pour faire fonctionner le projet puis nous détaillerons dans une seconde partie les différents bots et automatismes que nous avons découverts lors de ce cours. Nous expliciterons également les différentes étapes suivies pour la mise en place de certains de ces bots.

## Mise en place technique du projet

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

#### Actions éffectuées

#### Regard critique et conclusion

## Conclusion