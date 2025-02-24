---
title: Setup CI/CD Workflow with Github Actions to AWS
date: 2025-02-24 12:46:23 +/-TTTT
categories: [Github Actions]
tags: [github, aws, ci/cd]     # TAG names should always be lowercase
toc: true
comments: true
---

# Introduction

After a lot of time not publishing new content to my personal blog which I intent to store some of the work that I've done and learned throught the years, now finally it is time to get back to it and post some more content. If some how you are wondering why I stopped publishing more content it is very simple, I became father for the second time and I've been very busy with my new born :P .

Now for this article lets try to address the following:

First lets talk about Github Actions, this is an *Integration Delivery* tool very similar to bitbucket pipelines, jenkins, aws codedeploy, etc. It is very useful if you have your code stored in Github and you don't want to use other services to deploy your code to AWS for instance.

# Get to the point
Github Actions uses workflow files in YML format to handle all the steps that you want to do but lets sat you need to setup a Github Actions to handle CI/CD to more than 1 branch and you don't want to use more than 1 file in order to have a simple file that handles both scenarios.

We are going to use branches *stage* and *main* for this excercise.

```
name: Autodeploy to AWS

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

env:
  AWS_REGION: ${{ vars.AWS_REGION}}
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  ECR_REPOSITORY: ${{ vars.AWS_ECR_REPOSITORY }}
  DOCKER_IMAGE_NAME: ${{ vars.AWS_CONTAINER_NAME }}
  ECS_SERVICE: ${{ vars.AWS_ECS_SERVICE }}
  ECS_CLUSTER: ${{ vars.AWS_ECS_CLUSTER }}
  ECS_TASK_DEFINITION: ${{ vars.AWS_ECS_TASK_DEFINITION }}
  CONTAINER_NAME: ${{ vars.AWS_CONTAINER_NAME }}
  ENVIRONMENT: ${{github.event.inputs.releaseType}}

jobs:
  set-env-variable:
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

      - name: Use Variable
        run: |
          echo "The environment is: ${{ steps.set-env.outputs.ENVIRONMENT }}"

  build-and-publish:
    needs: set-env-variable
    runs-on: ubuntu-latest
    environment: ${{ needs.set-env-variable.outputs.environment }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: "Create env file"
        run: |
          touch .env
          cat <<EOF > .env
          ${{ secrets.ENV }}
          EOF
          cp .env public/.env

      - name: Setup Docker
        uses: docker/setup-buildx-action@v1

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-region: ${{ env.AWS_REGION }}
          aws-access-key-id: ${{ env.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ env.AWS_SECRET_ACCESS_KEY }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build, tag, and push image to Amazon ECR
        id: build-image
        env:
          ECR_REPOSITORY: ${{ env.ECR_REPOSITORY }}
          IMAGE_TAG: ${{ env.DOCKER_IMAGE_NAME }}
        run: |
          docker build -t $ECR_REPOSITORY:$IMAGE_TAG .
          docker tag $ECR_REPOSITORY:$IMAGE_TAG $ECR_REPOSITORY:latest
          docker push $ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REPOSITORY:latest
          echo "image=$ECR_REPOSITORY:latest" >> $GITHUB_OUTPUT

      - name: Deploy to ECS
        run: |
          aws ecs update-service --region ${{ env.AWS_REGION }} --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment
```


:wink:

I Hope you have enjoyed this post and if it is useful to you please invite me a coffee to keep posting more... 


<script type="text/javascript" src="https://cdnjs.buymeacoffee.com/1.0.0/button.prod.min.js" data-name="bmc-button" data-slug="paulogue" data-color="#FFDD00" data-emoji="☕" data-font="Cookie" data-text="Buy me a coffee" data-outline-color="#000000" data-font-color="#000000" data-coffee-color="#ffffff" ></script>