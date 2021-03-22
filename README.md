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
3. Create a new image repo using AWS console or CLI with the following command:

    `aws ecr create-repository --repository-name quickstart-clamav --image-tag-mutability IMMUTABLE --image-scanning-configuration scanOnPush=true`

    - Change the repo name if required. Default is **quickstart-clamav**

## 2. Initial Configuration

1. [Fork this repo](https://guides.github.com/activities/forking/) into your own GitHub account 
2. Run `git clone` to download the repo locally
3. Update the repo URL in **template.yml** at line 154
4. Update the image repo name in **buildspec.yml** on lines 21 and 22
5. [Create a personal access token from GitHub](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) 
   -  Under scopes, select **repo** - full control of private repositories
   -  Make sure to copy your personal access token upon creation
6. [Configure CodeBuild to access your GitHub repo](https://docs.aws.amazon.com/codebuild/latest/userguide/access-tokens.html)
   - [Example configuration settings](#example-codebuild-configuration)
7. Store your token in [AWS SecretsManager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
   - Take note of the secret name and key
8. Replace the secret name and key on line 121 in **template.yml**
   - **Example:** Token: '{{resolve:secretsmanager:`mysecret`:SecretString:`string`}}' 
   - Replace the **mysecret** and **string** sections, as shown above.

## 3. SAM Setup

1. Run `sam build` from the project home folder
   - [Click here for SAM build documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-build.html)
2. Run `sam deploy -g --capabilities CAPABILITY_NAMED_IAM` and fill out the prompts
   - You need URL of the image repo you created in the prerequisites
     - Example: `ACCOUNT_ID`.dkr.ecr.`AWS_REGION`.amazonaws.com/`REPO_NAME`
   - This solution deletes infected files by default
     - If you want to tag files instead, enter `Tag` as the value for the **PreferredAction** parameter
3. After the stack is deployed, [go to the CodeBuild Console](https://console.aws.amazon.com/codesuite/codebuild/projects) 
4. Once in the console, open the CodeBuild project [and add a VPC](https://docs.aws.amazon.com/codebuild/latest/userguide/vpc-support.html)

## Example CodeBuild Configuration

- Source:
  - GitHub
- Environment:
  - Environment Image: Managed Image
  - Operating System: Amazon Linux 2
  - Runtime: Standard
  - Image: standard:3.0
  - Image Version: Always use the latest image
  - Service Role: New service role
  - Role name: codebuild-Quickstart-ClamAV-service-role
- Build Specifications:
  - Use a buildspec file

# Planned Future Enhancements:

1. Functional enhancements:
    - Make this solution work with one or more existing buckets
    - Input VPC to allow codebuild to reach internet and avoid manual modification of codebuild project
    - Add support for S3 object versions
2. Security:
    - Trim down the IAM permissions across all the roles in the solution
3. Internal Operations (In QuickStart account?):
    - Build processes to produce CloudFormation templates for Lambda, Fargate and EC2
    - Build process to update container image nightly in the public repo
4. Prepare Cost estimations and Licenses
5. Document QuickStart guide for customers
