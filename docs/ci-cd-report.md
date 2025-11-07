# BobApp CI/CD Overview

## Workflow Stages
- **Triggering**: Runs on every pull request and on pushes to `main`/`master`. Deployments only happen on `main` pushes.
- **Backend quality**: Sets up Temurin JDK 11, runs `mvn verify` to execute unit tests and generate JaCoCo coverage, then preserves reports as workflow artifacts.
- **Frontend quality**: Uses Node.js 16, runs `npm ci` and headless Angular unit tests (`ng test --code-coverage`) to produce an `lcov.info` coverage report tailored for CI.
- **Static analysis**: Executes the SonarQube scanner against both modules, enforces the configured quality gate, and fails the job if KPIs are not met.
- **Container delivery**: After a successful quality stage, builds Docker images for the back-end and front-end via Buildx and pushes versioned (`latest`, commit SHA) tags to Docker Hub.

## Tooling & Configuration
- GitHub Actions workflow lives in `.github/workflows/ci-cd.yml`, orchestrating tests, quality checks, and container publishing.
- `sonar-project.properties` centralises analysis settings for both modules (coverage paths, TypeScript config, exclusions). Update the `sonar.projectKey` and `sonar.organization` values to match your Sonar instance.
- Front-end Karma config now emits `lcov` reports and runs Chrome in headless CI mode (see `front/karma.conf.js`).
- Docker build uses Node 16 LTS to stay compatible with Angular 14 while nginx 1.25 serves the production bundle (`front/Dockerfile`). The back-end image is produced from `maven:3.6.3-jdk-11-slim` and runs on `openjdk:11-jdk-slim` (`back/Dockerfile`).

## Proposed KPIs
- **Minimum overall code coverage ≥ 60 %** (short term). Raise the threshold as the project gains additional tests. Enforce via Sonar quality gate using the JaCoCo and lcov reports.
- **New blocker issues = 0 on every Sonar analysis**. Pipeline fails if Sonar detects any blocker-level findings on new code.
- Optional medium-term KPI: keep Sonar maintainability and reliability ratings at **A**, and track build success rate ≥ 95 % across rolling 30 days.

## Current Metrics Snapshot
- Backend line coverage: **37.8 %** (17/45 lines) from the latest local JaCoCo report in `back/target/site/jacoco/jacoco.xml`. Key gaps are in `Joke` model and `JsonReader` helper classes.
- Front-end coverage will be generated automatically from `front/coverage/bobapp/lcov.info` once the workflow runs under Node.js 16 in GitHub Actions. Review the uploaded artifact and Sonar dashboard after the first CI execution.
- Sonar quality gate status will be visible inside the GitHub Checks tab (`SonarQube Quality Gate`) once credentials (`SONAR_TOKEN`, `SONAR_HOST_URL`) are configured and a run completes.

## User Feedback Analysis
- **“Impossible de poster une suggestion de blague”**: indicates a regression in the suggestion form (likely front-end spinner never resolves). Prioritise reproducing this flow and add an end-to-end test guarding the API call.
- **“Bug sur le post de vidéo toujours présent”**: recurring defect signals absence of regression tests and poor triage. Add issue tracking with labels/priorities and cover the fix with integration tests touching media upload.
- **“Ça fait une semaine que je ne reçois plus rien”**: points to notification delivery outages. Investigate back-end scheduling/jobs and add monitoring (e.g., heartbeat metrics, alerting) to surface failures quickly.
- **“J'ai supprimé ce site de mes favoris”**: general dissatisfaction; focus on stabilising core flows above and communicate roadmap updates publicly (changelog, status page) to regain trust.

## Immediate Next Steps
- Register the project in Sonar (cloud or self-hosted), set secrets (`SONAR_TOKEN`, `SONAR_HOST_URL`, `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`) and run the workflow to establish the baseline dashboard.
- Analyse the first Sonar report to calibrate quality gate thresholds (e.g., enforce code smells, duplication ceilings) and refine KPIs if needed.
- Expand automated tests to raise coverage toward the 60 % target, beginning with untested back-end models/services and the failing user journeys highlighted above.
