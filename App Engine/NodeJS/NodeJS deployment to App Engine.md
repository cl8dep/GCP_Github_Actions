NodeJS deployment to App Engine
=========================================================

## Requirements
For use this pipeline is important have two secrets:
- PROJECT_ID: With the project id of GCP.
- GCP_SA_JSON_KEY: With the file content of the JSON Service Account credentials.

So the app.yaml file must be in the project root. This file act like a manifest of Cloud Run deployment.

## Github Action Workflow
```yaml
name: Deployment

on:
  push:
    branches:
      - master

env: 
  SERVICE_NAME: sample-backend

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@master

    - name: Set up Node.js version
      uses: actions/setup-node@v1
      with:
        node-version: '14.x'

    - name: Install dependencies
      run: npm install

    - name: Build
      run: npm run build    

    - name: Setup environment vars
      env:
        SERVICE_NAME: ${{ env.SERVICE_NAME }}
      run: eval "echo \"$(cat app.yaml)\"" > app.yaml
    
    - name: Set up GCloud
      uses: google-github-actions/setup-gcloud@master
      with:
        version: '290.0.1'
        service_account_key: ${{ secrets.GCP_SA_JSON_KEY }}
        export_default_credentials: true

    - name: Deploy
      uses: google-github-actions/deploy-appengine@main
      with:
        project_id: ${{ secrets.PROJECT_ID }}

```