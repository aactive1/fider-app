# Fider Application — Docker & CI/CD

This repository contains the application and container configuration for my AWS EKS DevOps portfolio project.

[Fider](https://github.com/getfider/fider) is an open-source customer feedback platform where users can submit ideas, vote on suggestions and discuss feature requests.

I used Fider as an existing application to focus on **containerisation, local development, CI/CD automation and secure container image delivery to AWS**.

## What I implemented

- **Docker:** Created a multistage Dockerfile to build the Go backend and JavaScript frontend, using a smaller runtime image.
- **Container security:** Configured the application to run as a non-root user and added a health check.
- **Docker Compose:** Set up Fider, PostgreSQL and MailHog for local development.
- **Persistent storage:** Used a Docker volume to preserve PostgreSQL data between container restarts.
- **GitHub Actions:** Automated application checks, Docker image builds and security scanning.
- **Trivy:** Integrated secret scanning and container vulnerability scanning into the CI pipeline.
- **Amazon ECR:** Used GitHub Actions and AWS OIDC authentication to publish images tagged with their Git commit SHA.

## CI/CD workflow

```text
GitHub Push / Pull Request
          |
     GitHub Actions
          |
    Application Checks
          |
      Trivy Scans
          |
    Build Docker Image
          |
      Amazon ECR
          |
   Update Image Tag
     in fider-gitops
          |
        Argo CD
          |
       Amazon EKS
```

The pipeline runs application checks, builds a Docker image and performs security scanning.

Images are published to Amazon ECR using GitHub Actions and AWS OIDC authentication, avoiding long-lived AWS access keys.

Application deployment is managed separately through the GitOps repository, where image promotion between environments is controlled manually.

## Run locally with Docker Compose

### Prerequisites

- Docker Desktop or Docker Engine with Docker Compose
- Git

### 1. Clone the repository

```bash
git clone https://github.com/aactive1/fider-app.git
cd fider-app
```

### 2. Configure environment variables

Copy the example environment file:

```bash
cp .env.example .env
```

Open `.env` and configure the application secret and PostgreSQL credentials.

Example configuration:

```dotenv
BASE_URL=http://localhost:3000
GO_ENV=development
JWT_SECRET=<YOUR_RANDOM_SECRET>

POSTGRES_USER=fider
POSTGRES_PASSWORD=<YOUR_DATABASE_PASSWORD>
POSTGRES_DB=fider

DATABASE_URL_CONTAINER=postgres://fider:<YOUR_DATABASE_PASSWORD>@postgres:5432/fider?sslmode=disable
```

Replace the placeholders with your own values.

The PostgreSQL password must match the password in `DATABASE_URL_CONTAINER`. Special characters in the connection URL must be URL-encoded.

Docker Compose uses the service name `postgres` to connect Fider to the database.

The `.env` file is excluded from Git and should not be committed.

### 3. Build and start the containers

```bash
docker compose up -d --build
```

Docker Compose starts three services:

| Service | Purpose | Address |
|---|---|---|
| Fider | Customer feedback application | http://localhost:3000 |
| PostgreSQL | Application database | Internal Docker network |
| MailHog | Local email testing | http://localhost:8025 |

### 4. View application logs

```bash
docker compose logs -f fider
```

### 5. Stop the containers

```bash
docker compose down
```

The PostgreSQL data remains stored in the Docker volume.

To remove the containers and delete the local database data:

```bash
docker compose down -v
```

## Project structure

```text
fider-app/
├── .github/workflows/    # CI/CD pipelines
├── Dockerfile            # Multistage container build
├── docker-compose.yml    # Local development services
├── .env.example          # Environment configuration
├── app/                  # Application source code
├── migrations/           # Database migrations
├── package.json          # JavaScript dependencies
├── go.mod                # Go dependencies
└── README.md
```

## Related repositories

- [fider-infra](https://github.com/aactive1/fider-infra) — AWS infrastructure provisioned using Terraform.
- [fider-gitops](https://github.com/aactive1/fider-gitops) — Kubernetes deployments, Helm, Argo CD and monitoring.

## Acknowledgements

Fider is an open-source application developed by the [Fider project](https://github.com/getfider/fider).

My contributions focus on containerisation, the local development environment, CI/CD automation and integration with the AWS EKS infrastructure.

The original application remains subject to its open-source licence.
