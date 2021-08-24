Cloud Storage Bucket deployment Static Web App
=========================================================

## Requirements
For use this pipeline is important have one secret:
- GCP_SA_JSON_KEY: With the file content of the JSON Service Account credentials.


## Github Action Workflow
```yml
name: Deployment

on:
  push:
    tags:
      - 'v*'

env:
#Bucket name
  BUCKET: gs://sample-bucket

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master

    - name: Set up Node.js version
      uses: actions/setup-node@v1
      with:
        node-version: '12.x'

    - name: Install dependencies
      run: npm install  

    - name: Build
      run: npm run build

    - name: setup-gcloud
      uses: google-github-actions/setup-gcloud@master
      with:
        version: '290.0.1'
        service_account_key: ${{ secrets.GCP_SA_KEY }}
        export_default_credentials: true

    - name: upload-file
      run: gsutil -h "Cache-Control:public, max-age=900" cp -r build/* ${{ env.BUCKET }}
```