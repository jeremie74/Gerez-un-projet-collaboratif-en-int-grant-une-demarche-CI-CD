# BobApp

Clone project:

> git clone XXXXX

## Front-end 

Go inside folder the front folder:

> cd front

Install dependencies:

> npm install

Launch Front-end:

> npm run start;

### Docker

Build the container:

> docker build -t bobapp-front .  

Start the container:

> docker run -p 8080:8080 --name bobapp-front -d bobapp-front

## Back-end

Go inside folder the back folder:

> cd back

Install dependencies:

> mvn clean install

Launch Back-end:

>  mvn spring-boot:run

Launch the tests:

> mvn clean install

### Docker

Build the container:

> docker build -t bobapp-back .  

Start the container:

> docker run -p 8080:8080 --name bobapp-back -d bobapp-back 

## CI/CD pipeline

The repository ships with a GitHub Actions workflow (`.github/workflows/ci-cd.yml`) that:

- runs Maven and Angular unit tests with coverage (JaCoCo + lcov);
- triggers a SonarQube/SonarCloud scan and enforces the quality gate;
- builds and pushes Docker images (front/back) to Docker Hub on pushes to `main`.

### Required secrets

Configure these repository secrets before running the workflow:

- `SONAR_HOST_URL` – URL of your SonarQube/SonarCloud instance (for SonarCloud use `https://sonarcloud.io`);
- `SONAR_TOKEN` – analysis token generated from Sonar;
- `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` – Docker Hub credentials with push rights.

Update `sonar-project.properties` with your own `sonar.projectKey` and `sonar.organization`.

Additional documentation: `docs/ci-cd-report.md`.
