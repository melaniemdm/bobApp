## Présentation du Workflow CI/CD – Projet Angular + Spring Boot

### **Objectif**

Mettre en place une CI/CD avec tests, analyse de qualité SonarQube et déploiement d’images Docker du projet fullstack bobApp.


### **Outils**

| Outil | Utilité principale |
| --- | --- |
| Github actions | Automatise les étapes d’intégration et de déploiement du projet en lançant des actions à chaque changement du code |
| SonarCloud | Analyse automatique de la qualité du code bobapp: détection de bugs, vulnérabilités, code smells, duplication, et suivi de la couverture de tests |
| Docker | Conteneurise de l’application Bobapp (backend et frontend) pour la rendre portable, isolée et facile à déployer sur tout autre environnement |
| Docker Hub | Permet de centraliser, partager et récupérer les images Docker prêtes à l’emploi pour automatiser les étapes de build, test ou déploiement dans un projet CI/CD |
| Jasmine | Framework de tests unitaires pour Angular. Permet de valider que le code front fonctionne comme prévu, sans erreur. |
| JaCoCo | Génère des rapports de couverture de code pour les tests Java. |
| Maven | Outil de build Java. Compile, exécute les tests, génère les rapports de couverture (JaCoCo) et lance l’analyse Sonar. |



### **Déclencheurs du Workflow**

| Type de déclencheur | Événement GitHub | Description | Objectif |
| --- | --- | --- | --- |
| Push | Sur la branche main | Exécute le workflow lors d’un push sur main | Automatiser la CI/CD après chaque modification du code en production |
| Pull Request | opened, synchronize, reopened | Déclenchement à l’ouverture ou mise à jour d’une Pull Request et la réouverture | Permet d’effectuer les tests et analyses avant fusion |


### **Job 1 _ build-and-analyze : Compilation, tests et analyse de code**

**1.1 Récupération du code source**

- **Action** : actions/checkout@v4

- **Paramètre** : fetch-depth: 0 pour que SonarQube accède à l'historique Git.

### **1.2 Backend – Spring Boot**


- **Installation Java 17** via actions/setup-java@v4

- **Commandes Maven** :

    - **mvn clean verify** => Compilation et exécution des tests

    - **Génération du rapport JaCoCo** (couverture de code)

- **Upload du rapport JaCoCo** comme artefact GitHub



### **Objectifs**

| Étape | Description | Objectif |
| --- | --- | --- |
| Checkout | Récupération du code source | Nécessaire pour que SonarQube accède à l’historique Git |
| Setup Java | Installation de Java 17 (Temurin) | Préparer l’environnement pour compiler le backend Spring Boot |
| Build + test backend | mvn clean verify dans ./back | Compiler, exécuter les tests unitaires et générer la couverture JaCoCo |
| Upload JaCoCo report | Upload du dossier de couverture | Rendre le rapport consultable dans GitHub |

### **1.3 Frontend – Angular**

- **Installation Node.js 20** via actions/setup-node@v4

- **Installation des dépendances** avec npm install

- **Exécution des tests Angular** avec couverture :

    - **Navigateur** : ChromeHeadless


    - **Génération** du fichier lcov.info

- **Upload de la couverture Angular** comme artefact


### **Objectifs**

| Étape | Description | Objectif |
| --- | --- | --- |
| Setup Node.js | Installation de Node.js v20 | Préparer l’environnement pour les tests Angular |
| Install frontend dependencies | npm install dans ./front | Installer toutes les dépendances nécessaires à l’exécution des tests |
| Run frontend tests with coverage | npm test avec --code-coverage | Exécuter les tests Angular en mode headless et générer lcov.info |
| Upload Angular coverage report | Upload du dossier coverage/ | Stocker le rapport de couverture Angular comme artefact |



### **Job 2 – Sonar-analysis: Analyse SonarQube**


### **1.4 Analyse SonarQube**

### **Objectifs**

| Partie analysée | Outil | Spécificités | Objectif |
| --- | --- | --- | --- |
| Backend | mvn sonar:sonar | Analyse Java + JaCoCo | Analyse statique du backend ( bugs, duplication, couverture) |
| Frontend | SonarSource/sonarqube-scan-action@v5 | Analyse TypeScript + couverture Angular (lcov.info) | Analyse statique du frontend (Angular) avec couverture à partir du fichier lcov.info |

### **Job 3 – docker-build-push : Construction et publication des images Docker**

### **2.1 Authentification Docker**

Utilisation de secrets GitHub DOCKER_USERNAME et DOCKER_TOKEN

### **2.2 Construction et Push des images**

| Composant | Chemin | Image Docker |
| --- | --- | --- |
| Backend | ./back | bobapp-backend:latest |
| Frontend | ./front | bobapp-frontend:latest |

### **Objectifs**

| Étape | Description | Objectif |
| --- | --- | --- |
| Checkout | Re-récupération du code source  | Assurer l’accès au code avant construction des images |
| Docker login | Connexion à Docker Hub avec secrets GitHub | Autoriser le push d’images vers le registre Docker Hub |
| Build & push backend image | Construction Docker de ./back | Produire une image Docker à jour du backend |
| Build & push frontend image | Construction Docker de ./front | Produire une image Docker à jour du frontend |

### **Résumé (Objectifs généraux par bloc)**

| Bloc | Objectif global |
| --- | --- |
| CI (build + test) | Vérifier automatiquement la qualité et stabilité du code |
| Coverage | Générer des rapports de couverture utiles pour la qualité logicielle |
| SonarQube | Assurer un suivi continu des métriques de qualité ( duplication, couverture...) |
| Docker | Automatiser la construction et la publication des images pour déploiement |


### **Proposition de KPIs qualité pour le projet bobapp**

### **KPI 1 – Couverture de code minimale**

| Indicateur | Valeur actuelle | Seuil recommandé | Pourquoi ? |
| --- | --- | --- | --- |
| Code Coverage (%) | 38.8% (back), 47.6% (front) | 80 % recommandé | Garantir qu’au moins 80 % du code est couvert par des tests pour limiter les régressions et améliorer la robustesse du projet |


### **KPI 2  – Note de fiabilité (Reliability)**

| Indicateur | Valeur actuelle | Seuil recommandé | Pourquoi ? |
| --- | --- | --- | --- |
| Reliability  (A => E) | D (backend),  A (frontend) | Minimum B | Cet indicateur mesure la présence de bugs potentiels dans le code. Un mauvais score (C, D ou E) indique un risque de défaillance fonctionnelle. |

### **Metrics actuels Backend**

| Indicateur | Valeur | Analyse |
| --- | --- | --- |
| Security | A (0) | Aucun problème de sécurité détecté |
| Reliability | D (1) | 1 bug détecté – priorité à corriger |
| Maintainability | A (8 smells) | Code propre dans l’ensemble, bien maintenable |
| Hotspots Reviewed | 0.0% | Aucun hotspot de sécurité n’a été examiné (à réaliser manuellement) |
| Coverage | 38.8% | Faible couverture – insuffisante pour garantir la robustesse |
| Duplications | 0.0% | Aucun code dupliqué  |



### **Conclusion**


 - Reliability : Corriger le bug bloquant identifié 
 - Hotspots Reviewed : Action manuel a réaliser
 - Coverage : Augmenter la couverture de tests (objectif  : 80 %)
 - Maintainability : Réduire  les code smells pour garder une base propre, objectif :  zéro code smell bloquant ou critique


### **Metrics actuels Frontend**

| Indicateur | Valeur | Analyse |
| --- | --- | --- |
| Security | A (0) | Aucun problème de sécurité détecté |
| Reliability | A (0) | Aucun bug détecté |
| Maintainability | A (5 smells) | Très bonne maintenabilité |
| Hotspots Reviewed | 100% | Tous les hotspots ont été examinés |
| Coverage | 47.6% | Couverture moyenne – doit être améliorée |
| Duplications | 0.0% | Aucun duplicata |

### **Conclusion**

- Coverage : Augmenter la couverture de tests (objectif minimum  : 80 %)

- Maintainability : Réduire davantage les code smells pour garder une base propre,  objectif :  zéro code smell bloquant ou critique


### **Retours utilisateurs pertinents**

- **Problème critique de publication**

« Impossible de poster une suggestion de blague, le bouton tourne et fait planter mon navigateur. »

-  **Bug persistant non corrigé**

« Bug sur le post de vidéo signalé il y a deux semaines, toujours présent. »

-  **Absence de support / réponse**

« J’ai envoyé un mail il y a 5 jours, toujours pas de réponse. »

-  **Perte de confiance utilisateur**

« J’ai supprimé ce site de mes favoris ce matin. »


**La CI/CD est une réponse organisationnelle qui :**

- réduit le temps de mise en production des correctifs de l’application,

- améliore la qualité du code,

- permet à l’équipe de se concentrer davantage sur l’utilisateur.








