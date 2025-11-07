# Rapport CI/CD BobApp

## 1. Étapes du workflow (fichier `.github/workflows/ci-cd.yml`)
1. **Déclencheurs** – Le pipeline démarre à chaque `push` sur `main`/`master` et sur toutes les Pull Requests. Cela garantit que les vérifications qualité tournent avant toute fusion.
2. **Qualité back-end** – Mise en place de Temurin JDK 11, exécution de `mvn -B verify` dans `back/` pour lancer les tests unitaires Spring Boot, générer la couverture JaCoCo (`back/target/site/jacoco/jacoco.xml`) et publier les rapports via `actions/upload-artifact`.
3. **Qualité front-end** – Installation de Node.js 16, `npm ci`, puis `npm run test -- --watch=false --code-coverage` dans `front/` pour produire `front/coverage/bobapp/lcov.info`. Le rapport est archivé comme artefact.
4. **Analyse SonarQube** – `SonarSource/sonarqube-scan-action` lit `sonar-project.properties`, agrège les rapports JaCoCo + lcov et applique le Quality Gate (`SonarSource/sonarqube-quality-gate-action`). Le pipeline échoue si les seuils Sonar ne sont pas respectés.
5. **Chaîne Docker** – Si les jobs qualité réussissent et que l’événement est un `push` sur `main`, `docker/build-push-action` construit deux images : `back/Dockerfile` (multi-stage Maven → `eclipse-temurin:11-jre-jammy`) et `front/Dockerfile` (build `node:16.20-bullseye`, runtime `nginx:1.25-alpine`). Chaque image est poussée sur Docker Hub avec les tags `latest` et `:${{ github.sha }}`.

## 2. KPIs proposés
- **Couverture de code minimale ≥ 60 %** (KPI principal). La qualité gate Sonar doit échouer si la couverture globale descend sous 60 % ou si la couverture sur nouveau code est < 80 %.
- **0 défaut bloquant et 0 vulnérabilité critique sur le nouveau code**. Toute alerte Sonar en criticité “Blocker”/“Critical” doit bloquer la pipeline.
- **Taux de succès de pipeline ≥ 95 % sur 30 jours glissants**. Toute rupture doit être suivie d’une analyse de cause racine et d’une action corrective.

## 3. Métriques & retours

### 3.1. Métriques collectées (exécutions locales 07/11/2025)
- **Back-end** (`back/target/site/jacoco/jacoco.xml`) : 38,64 % de couverture ligne (17/44) et 50,00 % de couverture branche après `mvn -B verify`. Les classes `JsonReader` et `Joke` restent non testées.
- **Front-end** (`front/coverage/bobapp/lcov.info`) : 83,33 % de couverture ligne, 76,92 % statements, 57,14 % fonctions après `npm run test -- --watch=false --code-coverage`. Les tests ont remonté de multiples erreurs “`mat-toolbar` n’est pas connu”, signe qu’Angular Material n’est pas correctement importé dans le `TestBed`.
- **Sécurité npm** : `npm ci` recense 44 vulnérabilités (9 faibles, 14 modérées, 17 élevées, 4 critiques). Aucune mitigation automatique n’a été appliquée.
- **Sonar** : en attente du premier run GitHub Actions avec `SONAR_TOKEN`/`SONAR_HOST_URL` pour publier la baseline.

### 3.2. Notes et avis (retours utilisateurs collectés)
- **“Impossible de poster une suggestion de blague”** – Incident critique sur le formulaire front, probablement lié à l’appel d’API indisponible. Priorité haute.
- **“Bug sur le post de vidéo toujours présent”** – Régression récurrente : absence de tests d’intégration couvrant l’upload vidéo. Priorité haute.
- **“Ça fait une semaine que je ne reçois plus rien”** – Rupture dans la diffusion de notifications/flux push. Priorité moyenne/haute, nécessite observabilité côté back-end.
- **“J’ai supprimé ce site de mes favoris”** – Sentiment négatif global : la stabilité et la communication produit sont perçues comme insuffisantes.

## 4. Recommandations
1. **Augmenter la couverture back-end** : ajouter des tests unitaires ciblant les services REST, la sérialisation `JsonReader` et les règles métier avant de monter le seuil Sonar à 60 %.
2. **Stabiliser les tests front** : importer les modules Angular Material (`MatToolbarModule`, `MatCardModule`, etc.) dans `app.component.spec.ts` afin d’éliminer les erreurs `NG0304` et fiabiliser la mesure de couverture.
3. **Traiter les vulnérabilités npm** : plan d’action `npm audit fix` + montée de version Angular/Material. Documenter les exceptions si une dépendance ne peut être corrigée immédiatement.
4. **Exploiter Sonar comme tableau de bord KPI** : configurer Quality Gate (couverture, duplications, code smells) et activer le check “Quality Gate” obligatoire sur GitHub avant fusion.
5. **Boucle de feedback** : transformer les retours utilisateurs ci-dessus en tickets suivis (label `P1`/`P2`), assortis de tests automatisés pour éviter les régressions futures.
