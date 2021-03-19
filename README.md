# README

## Architecture Diagram

![Architecture Diagram!](/QuickStart-ClamAV.png "Quick Start ClamAV")

## Runtime Architecture Flow
1. New objects are placed in S3 bucket in designated folder
2. S3 triggers the lambda function 
3. Lambda function pulls the latest docker image from ECR registry
4. Lambda function scans the new object for viruses using ClamAV open source

## Development Architecture Flow
- **A.** Developer checks in the code into GitHub repository
- **B.** GitHub WebHook triggers the CodeBuild build project
- **C.** CodeBuild build project packages the application into updated container image and uploads to ECR
- **D.** CodeBuild build project updates the lambda function to use latest image
- **E.** Timer Event runs every 24 hours and triggers the build. Build process will update the container image with latest virus definitions, publishes to ECR and updates the lambda function

## Pre-requisites
1. Install [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) & [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
1. Configure [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)
1. Create a new image repository using AWS console or CLI with command
    `aws ecr create-repository --repository-name quickstart-clamav --image-tag-mutability IMMUTABLE --image-scanning-configuration scanOnPush=true`

## How to deploy the solution
1. Fork this repository into your own git account and run `git clone` to download the code into your local computer
1. Update the repository URL in template.yml at line 154
1. Update the image repository name created above in buildspec.yml at both line 21 and 22
1. Create your personal access token from [github](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) 
    1. Learn more about configuring CodeBuild to access your source provider by [clicking here](https://docs.aws.amazon.com/codebuild/latest/userguide/access-tokens.html)
1. Store your token in AWS SecretsManager and note the secret name and key
1. Replace the secretname and key at line 121 in template.yml
1. Run `sam build` from the project home folder
1. Run `sam deploy -g --capabilities CAPABILITY_NAMED_IAM` and answer the questions. Note: You need URL of the image repository you created in pre-requisites. e.g., `ACCOUNT_ID`.dkr.ecr.`AWS_REGION`.amazonaws.com/`REPO_NAME`
1. Note that this solution deletes the infected files by default. If you want to just tag the file without deleting, ensure to enter _Tag_ as value for _PreferredAction_ parameter
1. After the stack is deployed, go to [AWS CodeBuild Console](https://console.aws.amazon.com/codesuite/codebuild/projects), open code build project and add VPC as per [instructions here](https://docs.aws.amazon.com/codebuild/latest/userguide/vpc-support.html)

## TODO:
1. Functional enhancements:
    1. Make this solution work with one or more existing bucket(s)
    1. Input VPC to allow codebuild to reach internet and avoid manual modification of codebuild project
    1. Add support for S3 object versions
1. Security:
    1. Trim down the IAM permissions across all the roles in the solution
1. Internal Operations (In QuickStart account?):
    1. Build processes to produce CloudFormation templates for lambda, fargate, EC2 versions
    1. Build process to update container image nightly in the public repo
1. Prepare Cost estimations and Licenses
1. Document QuickStart guide for customers
