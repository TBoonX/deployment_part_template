# deployment_part_template
This is the template for a generic deployment using the generic_deployment repository

A Github Action should be created in each fork of this repo in which a webhook is called to start the deployment on the server.
Here is an example: 

```
# Create container and push to repo

name: Publish Docker Image CI

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches:
      - testing
    paths-ignore: 
      - 'docs'
      - 'sql_scripts'
      - 'tests'

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
   push_to_registry:
    name: Push Docker image to GitHub Packages
    runs-on: ubuntu-latest
    steps:
      - name: Check out the repo
        uses: actions/checkout@v2
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1
      - name: Login to Docker Hub registry
        uses: docker/login-action@v1
        with:
          username: tboonx
          password: ${{ secrets.DOCKER_REPO_TBOONX_TOKEN }}
      - name: Push to Docker Hub
        uses: docker/build-push-action@v2
        with:
          context: .
          pull: true
          push: true
          tags:  tboonx/test:master
          file: ./docker/Dockerfile
      - name: Webhook for redeployment
        uses: zzzze/webhook-trigger@master
        with:
          data: "deploy=deployment_part_template"
          webhook_url: ${{ secrets.WEBHOOK_URL }}
          options: "-H \"Authorization: ${{ secrets.WEBHOOK_SECRET }}\""
```
