# README

## Architecture Diagram

![Architecture Diagram!](/QuickStart-ClamAV.png "Quick Start ClamAV")

## Runtime Architecture Flow

1. New objects are placed in S3 bucket in designated folder
1. S3 triggers the lambda function 
1. Lambda function pulls the latest docker image from ECR registry
1. Lambda function scans the new object for viruses using ClamAV open source

## Development Architecture Flow

**A.** Developer code into the GitHub repository

**B.** GitHub WebHook triggers the CodeBuild build project

**C.** CodeBuild build project packages the application into updated container image and uploads to ECR

**D.** CodeBuild build project updates the lambda function to use latest image

**E.** Timer Event runs every 24 hours and triggers the build. Build process will update the container image with latest virus definitions, publishes to ECR and updates the lambda function

# Deployment Guide

## 1. Prerequisites
1. Install [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) & [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
1. Configure [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)
1. Create a new image repository using AWS console or CLI with the following command:
   - Change the repository name below if required. Default is `quickstart-clamav`

    `aws ecr create-repository --repository-name quickstart-clamav --image-tag-mutability IMMUTABLE --image-scanning-configuration scanOnPush=true`

## 2. Initial Configuration

1. [Fork this repository](https://guides.github.com/activities/forking/) into your own GitHub account and run `git clone` to download the repo locally
1. Update the repository URL in **template.yml** at line 154
1. Update the image repository name in **buildspec.yml** on lines 21 and 22
1. [Create a personal access token from GitHub](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) 
1. [Configure CodeBuild to access your GitHub repo](https://docs.aws.amazon.com/codebuild/latest/userguide/access-tokens.html)
1. Store your token in [AWS SecretsManager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
   - Take note of the secret name and key
1. Replace the secret name and key on line 121 in **template.yml**

## 3. SAM Setup

1. Run `sam build` from the project home folder
   - [Click here for SAM build documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-build.html)
2. Run `sam deploy -g --capabilities CAPABILITY_NAMED_IAM` and fill out the prompts
   - You need URL of the image repository you created in the prerequisites
     - Example: `ACCOUNT_ID`.dkr.ecr.`AWS_REGION`.amazonaws.com/`REPO_NAME`
   - This solution deletes infected files by default
     - If you want to tag files instead, enter `Tag` as the value for the **PreferredAction** parameter
3. After the stack is deployed, [go to the CodeBuild Console](https://console.aws.amazon.com/codesuite/codebuild/projects) 
4. Once in the console, open the CodeBuild project [and add a VPC](https://docs.aws.amazon.com/codebuild/latest/userguide/vpc-support.html)

# Planned Future Enhancements:

1. Functional enhancements:
    - Make this solution work with one or more existing buckets
    - Input VPC to allow codebuild to reach internet and avoid manual modification of codebuild project
    - Add support for S3 object versions
1. Security:
    - Trim down the IAM permissions across all the roles in the solution
1. Internal Operations (In QuickStart account?):
    - Build processes to produce CloudFormation templates for Lambda, Fargate and EC2
    - Build process to update container image nightly in the public repo
2. Prepare Cost estimations and Licenses
3. Document QuickStart guide for customers
