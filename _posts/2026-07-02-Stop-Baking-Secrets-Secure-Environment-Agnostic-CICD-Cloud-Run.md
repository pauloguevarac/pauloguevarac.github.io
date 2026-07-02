---
title: "Stop Baking Secrets! Secure, Environment-Agnostic CI/CD for Google Cloud Run with GitHub Actions"
date: 2026-07-02 12:00:00 -0600
categories: [DevOps, Security]
tags: [github-actions, gcp, cloud-run, secrets, devsecops]
toc: true
comments: true
---

# Introduction

In modern serverless container deployments—such as **Google Cloud Run** or **AWS ECS**—speed and automation are often prioritized over security hygiene. A very common pattern in CI/CD pipelines is writing `.env` files and cloud credentials (like a Google Service Account JSON key) directly to the repository workspace right before running `docker build`. 

While this "works" and is easy to set up, it represents a **critical security anti-pattern**. 

Baking secrets and environment-specific configuration into your Docker image violates the core tenets of the [12-Factor App](https://12factor.net/) and exposes your infrastructure to severe risks.

In this article, we will analyze why baking secrets into Docker images is dangerous, how to build a completely **environment-agnostic** and **secure** container, and how to construct a robust, production-grade GitHub Actions CI/CD pipeline that deploys to Google Cloud Run utilizing keyless authentication and runtime secret injection.

---

## The Danger of Baked Secrets & Static Images

Let's look at what a vulnerable GitHub Actions deployment step typically looks like:

```yaml
# ❌ THE CRITICAL ANTI-PATTERN
- name: Setup Environment Variables
  run: |
    touch .env
    cat <<EOF >.env
    ${{ secrets.STAGING_ENV_VARIABLES }}
    EOF

- name: Setup GCP Service Account Key
  run: |
    touch sa-key.json
    cat <<EOF >sa-key.json
    ${{ secrets.GCP_SA_KEY }}
    EOF

- name: Build and push Docker image
  run: |
    docker build -t gcr.io/my-project/my-service:${{ github.sha }} .
    docker push gcr.io/my-project/my-service:${{ github.sha }}
```

### Why is this highly dangerous?

1. **Permanent Exposure in Docker Layers**: Docker images are constructed in layers. When you run `COPY . /app`, those sensitive files (`.env` and `sa-key.json`) are copied into the image. Even if you were to delete them in a later step (which most developers forget to do), the secrets remain permanently in the intermediate image layers. Anyone with read access to your private container registry (Google Artifact Registry, AWS ECR) can pull the image and easily unpack the layers to extract your database connection strings, API tokens, and service keys.
2. **Environment Lock-in**: Because the `.env` configuration is baked into the image, that specific Docker image is locked to a single environment (e.g., Staging). If you want to deploy the same code to Production, you must compile and build a brand-new image with the Production `.env` file. This completely violates the "Build Once, Deploy Many" philosophy of modern CI/CD. Rebuilding introduces the risk of dependency drift or subtle compilation differences between Staging and Production.
3. **Service Account Key Management Overhead**: Managing, rotating, and storing static JSON keys (`secrets.GCP_SA_KEY`) inside GitHub Secrets is an administrative headache and a major compliance risk. If a developer workstation or a GitHub repository is compromised, those long-lived keys can be leaked, giving attackers persistent access to your Google Cloud projects.

---

## The Solution: Fully Decoupled & Keyless Architecture

To achieve a secure, professional-grade CI/CD pipeline, we must completely decouple our secrets and configuration from the container build process. 

Our architecture will follow these three rules:
1. **Generic, Environment-Agnostic Docker Images**: The Docker image contains only the compiled code and dependencies. No `.env`, config, or JSON keys are present during the build.
2. **Keyless Runtime Service Authentication**: We do not provide key files to the application. Instead, we run our container under a dedicated Cloud Run runtime service identity (a Google Service Account). The application automatically fetches temporary IAM credentials from the instance metadata server.
3. **Runtime Secret Injection**: Secrets like database connection strings and third-party API tokens are securely stored in **Google Secret Manager** and mounted as environment variables at the container launch time.

```text
       [ Code Push ]
             │
             ▼
[ GitHub Actions Pipeline ]  ──( OIDC / WIF )──> Authenticates Keyless to GCP
             │
             ├──> Runs Linting, Tests, Syntax Check
             ├──> Builds Immutable, SECURE Docker Image (No Secrets!)
             └──> Pushes Image to Artifact Registry
             │
             ▼
    [ Deploy to Cloud Run ]
             │
             ├──> Mounts Secrets from Secret Manager at Runtime
             └──> Resolves IAM Permissions Keyless via Metadata Server (ADC)
```

---

## Step-by-Step Implementation

Let's build a highly secure and optimized GitHub Actions workflow for deploying a Python/Flask microservice (e.g., `video-dispatcher`) to Google Cloud Run.

### 1. The Secure GitHub Actions Workflow (`.github/workflows/autodeploy.yml`)

This production-grade workflow introduces:
* **Concurrency limits** to prevent race conditions during deployment.
* **A Quality Gate** (using Python syntax compilation and the ultra-fast `ruff` linter) that must pass before any build starts.
* **Workload Identity Federation (OIDC)** to authenticate with Google Cloud without long-lived JSON keys.
* **Complete removal** of secret-baking steps.
* **Runtime Secret Mapping** via Cloud Run's native Secret Manager integration.

```yaml
name: Secure Dispatcher Autodeploy

on:
  workflow_dispatch:
    inputs:
      releaseType:
        description: "Where to release (stage or prod)?"
        required: true
        default: "stage"
  push:
    branches:
      - stage
  release:
    types: [published]

# Prevent concurrent deployments of the same workflow on the same branch
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

env:
  PROJECT_ID: ${{ vars.GCP_PROJECT_ID }}
  REGION: ${{ vars.REGION }}
  REPOSITORY: video-dispatcher-repo
  SERVICE: video-dispatcher

jobs:
  lint-and-test:
    name: Lint & Syntax Check
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: "pip"

      - name: Install Linting Tools
        run: |
          python -m pip install --upgrade pip
          pip install ruff

      - name: Run Ruff Linter
        run: |
          # Ruff replaces flake8, black, and isort with lightning-fast execution
          ruff check .

      - name: Syntax Compilation Check
        run: |
          # Catch syntax errors before triggering the build process
          python -m py_compile main.py

  set-env-variable:
    name: Resolve Target Environment
    runs-on: ubuntu-latest
    outputs:
      environment: ${{ steps.set-env.outputs.ENVIRONMENT }}

    steps:
      - name: Set Environment Variable
        id: set-env
        run: |
          if [ "${{ github.event.inputs.releaseType }}" == "stage" ]; then
            echo "ENVIRONMENT=stage" >> $GITHUB_OUTPUT
          elif [ "${{ github.event.inputs.releaseType }}" == "prod" ]; then
            echo "ENVIRONMENT=prod" >> $GITHUB_OUTPUT
          elif [ "${{ github.event_name }}" == "push" ]; then
            echo "ENVIRONMENT=stage" >> $GITHUB_OUTPUT
          elif [ "${{ github.event_name }}" == "release" ]; then
            echo "ENVIRONMENT=prod" >> $GITHUB_OUTPUT
          else
            echo "ENVIRONMENT=stage" >> $GITHUB_OUTPUT
          fi

  build-and-deploy:
    name: Secure Build & Deploy
    needs: [lint-and-test, set-env-variable]
    runs-on: ubuntu-latest
    environment: ${{ needs.set-env-variable.outputs.environment }}

    steps:
      - name: Post Startup Notification to Slack
        uses: slackapi/slack-github-action@v2.1.0
        with:
          method: chat.postMessage
          token: ${{ secrets.SLACK_BOT_TOKEN }}
          payload: |
            {
              "channel": "${{ vars.SLACK_CHANNEL_ID }}",
              "text": ":rocket: Workflow *${{ github.workflow }}* started for `${{ needs.set-env-variable.outputs.environment }}` on `${{ github.ref_name }}`"
            }

      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Authenticate to Google Cloud (Keyless OIDC)
        uses: google-github-actions/auth@v2
        with:
          # Modern Workload Identity Federation (WIF) eliminates the need for any GCP SA JSON Key in GitHub Secrets!
          workload_identity_provider: ${{ secrets.GCP_WIF_PROVIDER }}
          service_account: ${{ secrets.GCP_WIF_SERVICE_ACCOUNT }}

      - name: Set up Google Cloud SDK
        uses: google-github-actions/setup-gcloud@v2

      - name: Configure Docker to use Artifact Registry
        run: |
          gcloud auth configure-docker ${{ env.REGION }}-docker.pkg.dev

      # -----------------------------------------------------------------
      # SECURITY NOTE:
      # No ".env" generation, and no Service Account JSON creation here!
      # The workspace remains clean. Image build is secure.
      # -----------------------------------------------------------------

      - name: Build and Push Docker Image
        run: |
          IMAGE="${{ env.REGION }}-docker.pkg.dev/${{ env.PROJECT_ID }}/${{ env.REPOSITORY }}/${{ env.SERVICE }}:${{ github.sha }}"
          docker build -t $IMAGE .
          docker push $IMAGE

      - name: Create or Update Google Cloud Run Service
        run: |
          IMAGE="${{ env.REGION }}-docker.pkg.dev/${{ env.PROJECT_ID }}/${{ env.REPOSITORY }}/${{ env.SERVICE }}:${{ github.sha }}"
          ENV_NAME="${{ needs.set-env-variable.outputs.environment }}"
          
          # DEPLOYMENT SPEC:
          # 1. We leverage GCP Secret Manager at runtime via `--set-secrets`.
          #    Secrets are fetched dynamically by the Cloud Run instance on startup.
          # 2. We inject environment-specific static variables dynamically via `--update-env-vars`.
          # 3. No service account key file is required inside the container! The application 
          #    runs under the Cloud Run service's identity, which is granted IAM permissions (ADC).
          
          gcloud run deploy ${{ env.SERVICE }} \
            --image $IMAGE \
            --region ${{ env.REGION }} \
            --platform managed \
            --memory 512Mi \
            --cpu 1 \
            --allow-unauthenticated \
            --port 80 \
            --set-secrets="MONGO_URI=MONGO_URI_${ENV_NAME}:latest,GRAPHQL_API_TOKEN=GRAPHQL_API_TOKEN_${ENV_NAME}:latest" \
            --update-env-vars="GCP_PROJECT_ID=${{ env.PROJECT_ID }},MONGO_DB_NAME=${{ vars.MONGO_DB_NAME }},MONGO_COLLECTION_NAME=${{ vars.MONGO_COLLECTION_NAME }},DEFAULT_FALLBACK_TOPIC=${{ vars.DEFAULT_FALLBACK_TOPIC }}"

  notify:
    name: Notify Status
    needs: build-and-deploy
    if: always()
    runs-on: ubuntu-latest

    steps:
      - name: Send status notification to Slack
        uses: slackapi/slack-github-action@v2.1.0
        with:
          method: chat.postMessage
          token: ${{ secrets.SLACK_BOT_TOKEN }}
          payload: |
            {
              "channel": "${{ vars.SLACK_CHANNEL_ID }}",
              "text": "${{ job.status == 'success' && ':white_check_mark:' || ':x:' }} Secure Workflow *${{ github.workflow }}* *${{ job.status }}* in `${{ github.repository }}`"
            }
```

---

### 2. How the Python Application Adapts

A common concern is: *"If I remove the `.env` file and the Service Account JSON file, how does my Python code authenticate and read these secrets?"*

The answer is **Application Default Credentials (ADC)** and direct system environment variable lookup.

#### Authenticaton (GCP Client Libraries)
When your Python app initializes a Google Cloud client (such as Pub/Sub, BigQuery, or Storage), the Google SDK automatically looks up credentials in a hierarchical order. 
If the `GOOGLE_APPLICATION_CREDENTIALS` environment variable is **not** set, the SDK queries the local Metadata Server. On Cloud Run, this server automatically returns short-lived IAM tokens of the service's execution identity.

No JSON key file, no `GOOGLE_APPLICATION_CREDENTIALS` config—it is entirely keyless and automatic!

```python
import logging
from google.cloud import pubsub_v1

# Keyless initialization! 
# The SDK automatically uses Cloud Run's execution service account identity.
try:
    publisher_client = pubsub_v1.PublisherClient()
    logging.info("Pub/Sub Publisher client initialized securely via ADC.")
except Exception as e:
    logging.critical(f"Failed to initialize Pub/Sub Client: {e}")
```

#### Secret Resolution
Because Google Cloud Run mounts secrets from Secret Manager directly into environment variables at launch time (e.g., using `--set-secrets="MONGO_URI=MONGO_URI_stage:latest"`), they are available to your application via standard system variables. 

Your Python code does not need custom integration with GCP APIs. It can continue using simple standard library calls:

```python
import os
from pymongo import MongoClient

# Read directly from environment variables.
# Whether injected via local .env in development, or Google Secret Manager in production.
MONGO_URI = os.getenv("MONGO_URI")
MONGO_DB_NAME = os.getenv("MONGO_DB_NAME", "default-db")

if MONGO_URI:
    mongo_client = MongoClient(MONGO_URI)
    db = mongo_client[MONGO_DB_NAME]
    logging.info("MongoDB connection established.")
else:
    logging.error("MONGO_URI is missing from environment variables!")
```

---

## Conclusion: The Benefits

By implementing this decoupled, keyless architecture in your CI/CD workflow, you achieve:

1. **Enhanced Security Posture**: No secrets or credential keys are baked into the container image or written to disk.
2. **Zero Key Management**: By adopting OIDC (WIF) in GitHub Actions and ADC inside Cloud Run, you completely eliminate static GCP Service Account JSON keys. There are no keys to expire, rotate, or leak.
3. **Environment-Agnostic Images (Build Once, Deploy Many)**: You can take the exact same Docker image compiled on staging, promote it to production, and launch it with different Secret Manager mappings and environment variables. This guarantees that what was tested in staging is *exactly* what runs in production.
4. **Faster Builds and Caching**: The Docker build step does not depend on secret injection, allowing Docker to optimize layer caching more effectively and reducing build times.

True DevSecOps maturity is about shifting security left while maintaining speed and immutability. Decoupling your secrets and transitioning to a keyless setup is one of the highest-impact improvements you can make to your cloud native applications.
