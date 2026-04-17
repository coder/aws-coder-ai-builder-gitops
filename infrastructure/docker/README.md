# Coder CodeBuild Container Image

Pre-baked container image for the Coder EKS deployment CodeBuild job.

## What's included

Base image: `aws/codebuild/amazonlinux2-x86_64-standard:5.0` (same as the original BuildProject)

| Tool       | Install method (mirrors pre_build)                |
|------------|---------------------------------------------------|
| AWS CLI    | `pip3 install --upgrade --user awscli`            |
| kubectl    | kubernetes-release/stable.txt (latest)            |
| eksctl     | weaveworks/eksctl latest tarball                  |
| Helm       | helm v3.16.4 tarball (configurable via build arg) |
| Coder CLI  | coder.com/install.sh --method standalone          |
| Terraform  | hashicorp/terraform latest zip from GitHub API    |

Python 3.12, git, curl, unzip, jq are already in the base image.

## Build args

Override at build time to pin versions:

```
docker build \
  --build-arg HELM_VERSION=3.16.4 \
  --build-arg CODER_VERSION=2.30.2 \
  -t coder-codebuild:latest .
```

## Deploying the pipeline

1. Zip the `docker/` directory contents and upload to S3:
   ```bash
   cd infrastructure
   zip -r codebuild-image-source.zip docker/
   aws s3 cp codebuild-image-source.zip s3://YOUR_BUCKET/codebuild-image-source.zip
   ```

2. Deploy the CloudFormation stack (the ECR Public alias is auto-discovered — no need to know it upfront):
   ```bash
   aws cloudformation deploy \
     --template-file codebuild_image_pipeline.yaml \
     --stack-name coder-codebuild-image \
     --parameter-overrides \
       SourceBucket=YOUR_BUCKET \
     --capabilities CAPABILITY_NAMED_IAM
   ```

3. Grab the image URI from the stack outputs:
   ```bash
   aws cloudformation describe-stacks \
     --stack-name coder-codebuild-image \
     --query 'Stacks[0].Outputs'
   ```

4. Trigger the first build:
   ```bash
   aws codebuild start-build --project-name coder-codebuild-image-builder
   ```

## Using the image in coder_deployment.yaml

Replace the `Image` property in the BuildProject Environment with the
`ECRPublicRepositoryUri` output value:

```yaml
Image: public.ecr.aws/YOUR_ALIAS/coder-codebuild:latest
```

The pre_build phase then simplifies to just generating the cluster config — no more downloading binaries.
