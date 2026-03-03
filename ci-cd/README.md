# 🔁 CI/CD

> Frequently used patterns for CI/CD pipelines with GitHub Actions and GitLab CI.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [GitHub Actions](#github-actions)
- [GitLab CI/CD](#gitlab-cicd)
- [Common Pipeline Patterns](#common-pipeline-patterns)
- [Secrets & Environment Variables](#secrets--environment-variables)

---

## GitHub Actions

### Key Concepts

| Term | Description |
|------|-------------|
| **Workflow** | YAML file in `.github/workflows/` |
| **Job** | Set of steps running on a runner |
| **Step** | Individual task in a job |
| **Action** | Reusable unit (marketplace or custom) |
| **Runner** | Server that executes jobs |
| **Trigger** | Event that starts a workflow |

### Triggers

```yaml
on:
  push:
    branches: [main, develop]
    paths: ['src/**', '!docs/**']
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'           # nightly at 2am UTC
  workflow_dispatch:               # manual trigger
    inputs:
      environment:
        description: 'Target env'
        required: true
        default: 'staging'
  release:
    types: [created]
```

### Complete CI/CD Workflow

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Upload coverage
        uses: codecov/codecov-action@v4

  build:
    name: Build & Push Image
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha,prefix=
            type=raw,value=latest

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    environment: production

    steps:
      - uses: actions/checkout@v4

      - name: Set up kubectl
        uses: azure/setup-kubectl@v4

      - name: Configure kubeconfig
        run: echo "${{ secrets.KUBECONFIG }}" | base64 -d > ~/.kube/config

      - name: Deploy
        run: |
          kubectl set image deployment/myapp \
            app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          kubectl rollout status deployment/myapp
```

### Reusable Workflow

```yaml
# .github/workflows/reusable-deploy.yml
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
    secrets:
      KUBECONFIG:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to ${{ inputs.environment }}
        run: echo "Deploying to ${{ inputs.environment }}"
```

**Calling a reusable workflow:**

```yaml
jobs:
  deploy:
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: production
    secrets:
      KUBECONFIG: ${{ secrets.KUBECONFIG }}
```

### Useful Actions

```yaml
# Checkout
- uses: actions/checkout@v4
  with:
    fetch-depth: 0         # full history

# Cache
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

# Upload artifact
- uses: actions/upload-artifact@v4
  with:
    name: build
    path: dist/

# Download artifact
- uses: actions/download-artifact@v4
  with:
    name: build
    path: dist/

# Set output
- id: vars
  run: echo "sha=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT
- run: echo ${{ steps.vars.outputs.sha }}

# Matrix strategy
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node: [18, 20]
runs-on: ${{ matrix.os }}
```

---

## GitLab CI/CD

### `.gitlab-ci.yml` Structure

```yaml
# .gitlab-ci.yml

# Global variables
variables:
  DOCKER_DRIVER: overlay2
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

# Pipeline stages (executed in order)
stages:
  - build
  - test
  - package
  - deploy

# Global before_script
before_script:
  - echo "Starting pipeline for $CI_COMMIT_REF_NAME"

# Build stage
build:
  stage: build
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn package -DskipTests
  artifacts:
    paths:
      - target/*.jar
    expire_in: 1 hour
  cache:
    key: $CI_COMMIT_REF_SLUG
    paths:
      - .m2/

# Test stage (parallel jobs)
unit-tests:
  stage: test
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn test
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml

code-quality:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm run lint

# Package stage
docker-build:
  stage: package
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
  only:
    - main

# Deploy stage
deploy-staging:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config set-cluster k8s --server="$K8S_SERVER"
    - kubectl config set-credentials admin --token="$K8S_TOKEN"
    - kubectl config set-context default --cluster=k8s --user=admin
    - kubectl config use-context default
    - kubectl set image deployment/myapp app=$IMAGE_TAG
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - main

deploy-production:
  stage: deploy
  script:
    - ./deploy.sh production
  environment:
    name: production
    url: https://example.com
  when: manual          # requires manual trigger
  only:
    - main
```

### GitLab CI Variables

| Variable | Description |
|----------|-------------|
| `$CI_COMMIT_SHA` | Full commit SHA |
| `$CI_COMMIT_SHORT_SHA` | Short commit SHA |
| `$CI_COMMIT_REF_NAME` | Branch or tag name |
| `$CI_COMMIT_BRANCH` | Branch name |
| `$CI_PIPELINE_ID` | Pipeline ID |
| `$CI_JOB_NAME` | Job name |
| `$CI_REGISTRY` | GitLab container registry URL |
| `$CI_REGISTRY_IMAGE` | Image path in registry |
| `$CI_REGISTRY_USER` | Registry username |
| `$CI_REGISTRY_PASSWORD` | Registry password |
| `$CI_PROJECT_NAME` | Project name |
| `$CI_PROJECT_PATH` | `namespace/project` |

---

## Common Pipeline Patterns

### Semantic Versioning

```yaml
# GitHub Actions: tag on release
- name: Get version
  id: version
  run: echo "VERSION=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT

- name: Build
  run: docker build -t myapp:${{ steps.version.outputs.VERSION }} .
```

### Multi-environment Deploy

```yaml
# GitHub Actions: environment-specific deploy
deploy:
  strategy:
    matrix:
      environment: [dev, staging]
  environment: ${{ matrix.environment }}
  steps:
    - run: ./deploy.sh ${{ matrix.environment }}
```

---

## Secrets & Environment Variables

```bash
# GitHub CLI — set secrets
gh secret set MY_SECRET --body "value"
gh secret set MY_SECRET < secret.txt
gh secret list

# GitLab CLI — set CI/CD variables
glab variable set MY_VAR --value "value" --masked
glab variable list
```

---

[← Back to Home](../README.md)
