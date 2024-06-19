# RISTEK CSUI - Infra Workflows

A collection of standard workflows for RISTEK. The purpose of this repository is to provide reusable standards in order to have simple, streamlined, and fast deployments. 
A lot of these workflows are used in collaboration with deployment configurations on our central infra repository.

## Standard Service Repo Build and Deploy

There are workflow files in this repo which can be called by service repositories. The purpose is to provide a standard workflow for building and deploying images on Central Infra which can be called/reused on any service. They are formatted as `<registry-provider>-service-build.yml` where `registry-provider` is the container registry provider (e.g. `ecr` for Amazon ECR).

1. Create a deployment workflow file on your service
2. In the job you want to use the workflow in, simply use the workflow as such:

   ```yaml
   jobs:
     arbitrary_build_and_deploy_job:
       name: Build and Deploy Arbitrary Service
       uses: RistekCSUI/infra/.github/workflows/ecr-service-build.yml@main
       with:
         INSTANCE: Barato
         SERVICES: summerfest/summerfest-be-stg
         REGISTRY: 638207107223.dkr.ecr.ap-southeast-1.amazonaws.com
         REGISTRY_IMAGE: summerfest-be
         REGISTRY_USER: AWS
         PLATFORMS: linux/amd64
         IMAGE_TAG: latest
         AWS_REGION: ap-southeast-1
       secrets:
         GH_TOKEN: ${{ secrets.CENTRAL_INFRA_GH_TOKEN }}
         AWS_ECR_ACCESS_KEY: ${{ secrets.AWS_ECR_ACCESS_KEY }}
         AWS_ECR_SECRET_KEY: ${{ secrets.AWS_ECR_SECRET_KEY }}
   ```
   
3. The `with` section specifies the input that the workflow will use. The `secrets` section is for sensitive inputs such as access keys and tokens. The inputs and secrets may differ for each supported registry.
4. For the `secrets` section, `GH_TOKEN` will always be needed as this is used to trigger the Central Infra Deployment workflow from other repositories.

### How it works

1. The service repository calls the `*-service-build.yml` workflow. What triggers the workflow from the service repository is up to the repo maintainer.
2. The first step is to login to the container registry.
3. Once signed in, it will build and push the image into the registry,
4. After the image has been pushed successfully, it will trigger a deployment on Central Infra with the new image and specified configuration

### Inputs

**All Registries**

| Field | Description |
|---|---|
| INSTANCE | The name of instance you want to deploy the service on. You can check the available service and server instance mapping on `service_config.yml` on the Central Infra repo |
| SERVICES | Comma separated list of path of service directories to be deployed on Central Infra (e.g. "summerfest/summerfest-be-stg, system/caddy”). For a service repository, this will most likely be just one path which is for the service deployment itself. For example, `summerfest/summerfest-be-stg` for a Summerfest BE deployment on Staging. |
| PLATFORMS | Platforms to build the image for (e.g. linux/amd64) |
| IMAGE_TAG | The image tag. By convention `latest` is for single environments and staging deployments and `stable` is for production. |

**Amazon ECR** (`.github/workflows/ecr-service-build.yml`)

| Field | Description |
|---|---|
| REGISTRY | The container registry which will be used |
| REGISTRY_IMAGE | Name of the image to build |
| REGISTRY_USER | REGISTRY_USER |
| AWS_REGION | The AWS region for the registry |
  
**Docker Hub** (`.github/workflows/dockerhub-service-build.yml`)

| Field | Description |
|---|---|
| IMAGE | Name of the image to build |
| DOCKERHUB_USER | User account for the Docker Hub registry |

### Secrets

**All Registries**

| Field | Description |
|---|---|
| GH_TOKEN | A GitHub personal access token, used to trigger the Central Infra Deployment workflow from other repositories. |

**Amazon ECR** (`.github/workflows/ecr-service-build.yml`)

| Field | Description |
|---|---|
| AWS_ECR_ACCESS_KEY | An access key to push and pull to the Amazon ECR registry |
| AWS_ECR_SECRET_KEY | A secret key paired with the Amazon ECR access key |

**Docker Hub** (`.github/workflows/dockerhub-service-build.yml`)

| Field | Description |
|---|---|
| DOCKERHUB_TOKEN | The Docker Hub access token to push to the registry |
