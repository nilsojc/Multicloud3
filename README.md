<p align="center">
  <img src="assets/diagram.png" 
</p>
  
## ☁️ MultiCloud, DevOps & AI Challenge — Day 3 — Building and Automating CI/CD Pipeline to Test, Stage and Deploy our E-Commerce Application. ☁️

This is part of the third project of the Multicloud, Devops and AI Challenge!

In this project we will be build and automating our E-commerce application testings and deployments for production using AWS CodePipeline so that every time we push changes to our application they are built and deployed automatically. 


<h2>Environments and Technologies Used</h2>

  - Amazon Web Services
  - Github Codespaces
  - AWS CodePipeline
  - AWS CodeBuild
  - Docker
  - Amazon Elastic Container Registry
  
  
<h2>Key Features</h2>  

✅
✅
✅


<h2>Step by Step Instructions</h2>

***1. Repo configuration***


NOTE: Keep in mind this is for a Linux environment, check the AWS documentation to install it in your supported OS.


   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install


We then do `AWS configure` and enter our access and secret key along with the region. Output format set to JSON. With this command we will double check that our credentials are put in place for CLI:

```
aws sts get-caller-identity
```

We will then roceed with installing the Docker CLI and Docker in Docker (Github Codespaces Setup)

```
curl -fsSL https://download.docker.com/linux/static/stable/x86_64/docker-20.10.9.tgz -o docker.tgz \
tar -xzf docker.tgz \
sudo mv docker/docker /usr/local/bin/ \
rm -rf docker docker.tgz
```

`Ctrl + p` on Github Codespace > `Add Dev Container Conf files` > modify your active configuration > click on Docker (Docker-in-Docker)

![image](/assets/image1.png)



***2.  Clone Frontend from Previous Repo, Configure AWS CodePipeline and AWS Code Build***

In this step, we will clone the frontend of our CloudMart e-commerce application in order to start building the code and start deploying the changes.

```
gh repo clone nilsojc/multicloud2
```

Then, we wil create a new pipeline with AWS CodePipeline:

```
aws codepipeline create-pipeline \
    --pipeline-name cloudmart-cicd-pipeline \
    --role-arn arn:aws:iam::your-account-id:role/your-codepipeline-role \
    --pipeline-structure '{
        "stages": [
            {
                "name": "Source",
                "actions": [
                    {
                        "name": "SourceAction",
                        "actionTypeId": {
                            "category": "Source",
                            "owner": "ThirdParty",
                            "provider": "GitHub",
                            "version": "1"
                        },
                        "configuration": {
                            "Owner": "nilsojc",
                            "Repo": "multicloud3",
                            "Branch": "main",
                            "OAuthToken": "your-github-oauth-token"
                        },
                        "outputArtifacts": [
                            {
                                "name": "SourceOutput"
                            }
                        ],
                        "runOrder": 1
                    }
                ]
            },
            {
                "name": "Build",
                "actions": [
                    {
                        "name": "BuildAction",
                        "actionTypeId": {
                            "category": "Build",
                            "owner": "AWS",
                            "provider": "CodeBuild",
                            "version": "1"
                        },
                        "configuration": {
                            "ProjectName": "cloudmartBuild"
                        },
                        "inputArtifacts": [
                            {
                                "name": "SourceOutput"
                            }
                        ],
                        "outputArtifacts": [
                            {
                                "name": "BuildOutput"
                            }
                        ],
                        "runOrder": 1
                    }
                ]
            },
            {
                "name": "Deploy",
                "actions": [
                    {
                        "name": "DeployAction",
                        "actionTypeId": {
                            "category": "Deploy",
                            "owner": "AWS",
                            "provider": "CodeDeploy",
                            "version": "1"
                        },
                        "configuration": {
                            "ApplicationName": "cloudmartDeploy",
                            "DeploymentGroupName": "cloudmartDeployGroup"
                        },
                        "inputArtifacts": [
                            {
                                "name": "BuildOutput"
                            }
                        ],
                        "runOrder": 1
                    }
                ]
            }
        ]
    }' \
    --region us-east-1
```

Next up, we will be configuring AWS CodeBuild to Build the Docker Image:

```
aws codebuild create-project \
    --name cloudmartBuild \
    --source '{
        "type": "GITHUB",
        "location": "https://github.com/yourusername/multicloud3.git",
        "gitCloneDepth": 1,
        "buildspec": "buildspec.yml"
    }' \
    --artifacts '{
        "type": "NO_ARTIFACTS"
    }' \
    --environment '{
        "type": "LINUX_CONTAINER",
        "image": "aws/codebuild/amazonlinux2-x86_64-standard:4.0",
        "computeType": "BUILD_GENERAL1_SMALL",
        "privilegedMode": true,
        "environmentVariables": [
            {
                "name": "ECR_REPO",
                "value": "your-ecr-repo-uri"
            }
        ]
    }' \
    --service-role arn:aws:iam::your-account-id:role/your-codebuild-role \
    --region us-east-1
```

For the build specification, we will use the following buildspec.yml:


```
version: 0.2
phases:
  install:
    runtime-versions:
      docker: 20
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws --version
      - REPOSITORY_URI=$ECR_REPO
      - aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/l4c0j8h9
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$CODEBUILD_RESOLVED_SOURCE_VERSION
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker image...
      - docker push $REPOSITORY_URI:latest
      - docker push $REPOSITORY_URI:$CODEBUILD_RESOLVED_SOURCE_VERSION
      - export imageTag=$CODEBUILD_RESOLVED_SOURCE_VERSION
      - printf '[{\"name\":\"cloudmart-app\",\"imageUri\":\"%s\"}]' $REPOSITORY_URI:$imageTag > imagedefinitions.json
      - cat imagedefinitions.json
      - ls -l

env:
  exported-variables: ["imageTag"]

artifacts:
  files:
    - imagedefinitions.json
    - cloudmart-frontend.yaml
```

Then we will specify the deployment, with the following `buildspec.yml`:

```
version: 0.2

phases:
  install:
    runtime-versions:
      docker: 20
    commands:
      - curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.18.9/2020-11-02/bin/linux/amd64/kubectl
      - chmod +x ./kubectl
      - mv ./kubectl /usr/local/bin
      - kubectl version --short --client
  post_build:
    commands:
      - aws eks update-kubeconfig --region us-east-1 --name cloudmart
      - kubectl get nodes
      - ls
      - IMAGE_URI=$(jq -r '.[0].imageUri' imagedefinitions.json)
      - echo $IMAGE_URI
      - sed -i "s|CONTAINER_IMAGE|$IMAGE_URI|g" cloudmart-frontend.yaml
      - kubectl apply -f cloudmart-frontend.yaml
```



***3. Testing the CI/CD Pipeline***

1. **Make a Change on GitHub:**
    - Update the application code in the **`cloudmart-application`** repository.
    - File `src/components/MainPage/index.jsx` line 93
    - Commit and push the changes.
    
    ```bash
    git add -A
    git commit -m "changed to Featured Products on CloudMart"
    git push
    ```
    
2. **Observe the Pipeline Execution:**
    - Watch how CodePipeline automatically triggers the build.
    - After the build, the deployment phase should begin.
3. **Verify the Deployment:**
    - Check Kubernetes using **`kubectl`** commands to confirm the application update.


<h2>Conclusion</h2>

In this project, I learned how to 
