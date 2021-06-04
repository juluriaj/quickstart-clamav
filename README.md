# README

## Architecture Diagram

![Architecture Diagram!](/QuickStart-ClamAV.png "Quick Start ClamAV")

## Runtime Architecture Flow

1. New objects are placed in S3 bucket in designated folder
2. S3 triggers the lambda function 
3. Lambda function pulls the latest docker image from ECR registry
4. Lambda function scans the new object for viruses using ClamAV open source

## Development Architecture Flow

**A.** Developer uploads code into the GitHub repo

**B.** GitHub WebHook triggers the CodeBuild build project

**C.** CodeBuild build project packages the application into the updated container image and uploads to ECR

**D.** CodeBuild build project updates the lambda function to use latest image

**E.** Timer Event runs every 24 hours and triggers the build. Build process will update the container image with latest virus definitions, publishes to ECR and updates the lambda function

# Deployment Guide

## 1. Prerequisites
1. [Install the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) and the [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
2. [Configure the AWS CLI with your credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)
3. [Install and run Docker on your local machine](https://www.docker.com/products/docker-desktop)
4. Create a new image repo using AWS console or CLI with the following command:

    `aws ecr create-repository --repository-name quickstart-clamav --image-tag-mutability IMMUTABLE --image-scanning-configuration scanOnPush=true`

    - Change the repo name if required. Default is **quickstart-clamav**

## 2. Initial Configuration

1. [Fork this repo](https://guides.github.com/activities/forking/) into your own GitHub account 
1. Run `git clone` to download the repo locally
1. [Create a personal access token from GitHub](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) 
   -  Under scopes, select **repo** - full control of private repositories and **admin:repo_hook** - full control of repository hooks
   -  Make sure to copy your personal access token value upon creation
   -  [Click here](https://docs.aws.amazon.com/codebuild/latest/userguide/access-tokens.html) for more information on using other source providers with CodeBuild
1. Store your token in [AWS SecretsManager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
   - Take note of the secret name and key

## 3. SAM Setup and Deployment

1. Run `sam build` from the project home folder
   - [Click here for SAM build documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-build.html)
1. Run `sam deploy -g --capabilities CAPABILITY_NAMED_IAM` and fill out the prompts
   - You need URL of the image repo you created in the prerequisites
     - Example: `ACCOUNT_ID`.dkr.ecr.`AWS_REGION`.amazonaws.com/`REPO_NAME`
   - Input the image repo URL as the value for both **ECSREPO** parameter and **image-repository**
   - This solution deletes infected files by default
     - If you want to tag files instead, enter `Tag` as the value for the **PreferredAction** parameter
1. Add VPC to your CodeBuild project
    - After the stack is deployed, [go to the CodeBuild Console](https://console.aws.amazon.com/codesuite/codebuild/projects) 
    - Once in the console, open the CodeBuild project [and add a VPC](https://docs.aws.amazon.com/codebuild/latest/userguide/vpc-support.html)
    - Click Validate VPC Settings to confirm there is internet connectivity

# Notes

## Limitations
1. [Lambda function code can access a writable /tmp directory with 512 MB of storage](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html#images-reqs). Please consider these limits when deploying this solution.
2. Currently, this solution must be deployed to a public AWS Region. GovCloud is not supported yet.
3. By default, the ClamAV scanner will only scan objects uploaded to the newly created S3 bucket. Read the instructions below to configure it for an existing bucket.

## Setup with an Existing Bucket

1. Within the S3 console, navigate to the S3 bucket you want to configure
2. Create an event notification
3. Configure the event notification to trigger on **Put** and **Post**
   - This can be modified based on your use case 
4. Set the prefix as `Inbound/`
5. Set the destination as a Lambda function
6. Select your ClamAV Lambda function as the destination
