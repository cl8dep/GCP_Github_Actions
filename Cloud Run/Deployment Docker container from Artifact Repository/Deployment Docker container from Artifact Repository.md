Cloud Run deployment with Docker from Artifact Repository
=========================================================

## Requirements
For use this pipeline is important have two secrets:
- PROJECT_ID: With the project id of GCP.
- GCP_SA_JSON_KEY: With the file content of the JSON Service Account credentials.

So the service.yaml file must be in the project root. This file act like a manifest of Cloud Run deployment.


## Github Action Workflow
```yml
name: Cloud Run Deployment

on:
  push:
    branches:
      - master

env:
#Name of the service
  SERVICE_NAME: qa-sample-backend
#Region where the service will be located
  REGION: us-east1
#Name of the project. It's used to store related Docker images
  PROJECT_NAME: sample-backend
#Registry base from GCP
  REGISTRY_BASE: us-east1-docker.pkg.dev/${{ secrets.PROJECT_ID }}

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Inject slug/short variables
        uses: rlespinasse/github-slug-action@v3.x

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Login to GAR
        uses: docker/login-action@v1
        with:
          registry: ${{ env.REGISTRY_BASE }}
          username: _json_key
          password: ${{ secrets.GCP_SA_JSON_KEY }}
          
      - name: Build and push
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          push: true
          tags: ${{ env.REGISTRY_BASE }}/${{ env.PROJECT_NAME }}/${{ env.SERVICE_NAME }}:${{ github.sha }}

      - name: Image digest
        run: echo ${{ steps.docker_build.outputs.digest }}

  deploy:
    needs: ['build']
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Setup env
        env:
          IMAGE: ${{ env.REGISTRY_BASE }}/${{ env.PROJECT_NAME }}/${{ env.SERVICE_NAME }}:${{ github.sha }}
          SERVICE_NAME: ${{ env.SERVICE_NAME }}
          NODE_ENV: production
          REGION: ${{ env.REGION }}
          CPU: 1000m
          MEMORY: 256Mi
          maxScale: '20'
          minScale: '2'
          containerConcurrency: 80
          ingress: internal-and-cloud-load-balancing
          #ENV vars here. They are replaced in the service.yaml file with the same name with $ before.
        run: eval "echo \"$(cat service.yaml)\"" > service.yaml

      - id: deploy
        uses: google-github-actions/deploy-cloudrun@main
        with:
          metadata: service.yaml
          credentials: ${{ secrets.GCP_SA_JSON_KEY }}
          region: ${{ env.REGION }}

      - id: test_url
        run: curl "${{ steps.deploy.outputs.url }}"
```