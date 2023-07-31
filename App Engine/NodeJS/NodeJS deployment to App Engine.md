NodeJS deployment to App Engine
=========================================================

## Requirements
For use this pipeline is important have two secrets:
- GCP_WORKLOAD_IDENTITY_PROVIDER: With the project id of GCP.
- GCP_SERVICE_ACCOUNT: With the file content of the JSON Service Account credentials.
- GCP_SERVICE_ACCOUNT: With the file content of the JSON Service Account credentials.

So the app.yaml file must be in the project root. This file act like a manifest of Cloud Run deployment.

## Github Action Workflow
```yaml
name: Dev Deployment to App Engine

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: "read"
      id-token: "write"

    steps:
      - uses: actions/checkout@master

      - name: Set up Node.js version
        uses: actions/setup-node@v3
        with:
          node-version: "14.x"

      - name: Install dependencies
        run: npm install

      - name: Build
        run: npm run build

      - name: Setup environment vars
        env:
          REACT_APP_API_URL: ${{ secrets.API_URL_DEV }}
        run: eval "echo \"$(cat app.yaml)\"" > app.yaml

      - id: "auth"
        uses: google-github-actions/auth@v1
        with:
          workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
          service_account: ${{secrets.GCP_SERVICE_ACCOUNT}}

      - name: Deploy
        uses: google-github-actions/deploy-appengine@v1


```