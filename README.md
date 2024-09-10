#

# Setup Steps

## Clone the GitHub repository

Clone repository: https://github.com/javiercarreraruiz/containerlambdacicd-blog.git

## Create artifacts bucket

```
export $ARTIFACTS_BUCKET_NAME=containerlambdacicd-blog-artifacts
aws s3 mb s3://$ARTIFACTS_BUCKET_NAME
```

## Create ECR repository

```
aws ecr create-repository --repository-name lambda-from-container-image
```

## Create AWS CodeConnections connection

Go to: https://console.aws.amazon.com/codesuite/settings/connections
Choose you AWS region.
Click on "Create Connection"
Follow the instructions to create the connection to GitHub. Call it containerlambda-gh-connection
Choose the correct region if it changed on the Management Console and click on Connect
Go back to https://console.aws.amazon.com/codesuite/settings/connections and copy the ARN of the new connection

## Replace placeholders in template files

_IMPORTANT:_ You need to adapt the next environment variables AND you need to install the envsubst command in your machine

```
export ACCOUNT_ID=YOUR_AWS_12_DIGIT_ACCOUNT_NUMBER
export REGION_ID=SOMETHING_LIKE_eu-west-1
export ARTIFACTS_BUCKET_NAME=THE_BUCKET_YOU_CREATED_PREVIOUSLY
export GITHUB_CONNECTION_ARN=THE_AWS_CODECONNECTIONS_CONNECTION_ARN
export GITHUB_USERNAME=yourgithubname
export GITHUB_REPONAME=containerlambdacicd-blog


envsubst '${ACCOUNT_ID} ${REGION_ID}' < buildspec.yml.template > buildspec.yml
envsubst '${ARTIFACTS_BUCKET_NAME} ${GITHUB_CONNECTION_ARN} ${GITHUB_REPONAME} ${GITHUB_USERNAME} ${REGION_ID}' < pipeline.json.template > pipeline.json
envsubst '${ACCOUNT_ID} ${ARTIFACTS_BUCKET_NAME} ${GITHUB_CONNECTION_ARN} ${GITHUB_REPONAME} ${GITHUB_USERNAME} ${REGION_ID}' < project-config.json.template > project-config.json
envsubst '${ACCOUNT_ID} ${ARTIFACTS_BUCKET_NAME} ${GITHUB_CONNECTION_ARN} ${GITHUB_REPONAME} ${GITHUB_USERNAME} ${REGION_ID}' < create-roles.sh.template > create-roles.sh
envsubst '${ACCOUNT_ID} ${ARTIFACTS_BUCKET_NAME} ${GITHUB_CONNECTION_ARN} ${GITHUB_REPONAME} ${GITHUB_USERNAME} ${REGION_ID}' < scripts/create_or_update_lambda_function.sh.template > scripts/create_or_update_lambda_function.sh
```

## Create IAM roles

```
chmod +x create-roles.sh
./create-roles.sh*
```

## Create CodeBuild project and CodePipeline pipeline

```
aws codebuild create-project --cli-input-json file://project-config.json

aws codepipeline create-pipeline --cli-input-json file://pipeline.json
```
