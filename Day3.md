# MultiCloud, DevOps & AI Challenge - Day 3
## Part 1: CI/CD Pipeline Configuration

### Create a free account on GitHub and then create a new repository on GitHub called cloudmart
- ![Image1](https://github.com/user-attachments/assets/0d4002c2-2aa8-4bbc-910d-90812e543400)

- ![Image2](https://github.com/user-attachments/assets/9962e392-d2b3-40a1-b988-7dad5e9db403)



        cd challenge-day2/frontend
        <Run GitHub steps>

 Start by pushing the changes in the CloudMart application source code to GitHub
 
          git status
          git add -A
          git commit -m "app sent to repo"
          git push

  - ![Image3](https://github.com/user-attachments/assets/44eef492-a8f5-4d8e-92f0-2d1c643ad5ea)
  - ![Image4](https://github.com/user-attachments/assets/f1b50f4a-b291-4ff7-b056-1350946979a4)
  - ![Image5](https://github.com/user-attachments/assets/f4786825-1343-4cb7-909d-b7b55550e70e)

### **Configure AWS CodePipeline**

1. **Create a New Pipeline:**
    - Access AWS CodePipeline.
    - Start the 'Create pipeline' process.
    - Name: `cloudmart-cicd-pipeline`
    - Use the GitHub repository `cloudmart-application` as the source.
    - Add the 'cloudmartBuild' project as the build stage.
    - Add the 'cloudmartDeploy' project as the deployment stage.

- ![Image6](https://github.com/user-attachments/assets/857ad27c-a18f-4a43-b82d-cb49ad8e2bd9)
- ![Image7](https://github.com/user-attachments/assets/6348e94d-c77c-4006-af1a-cfab09280a99)


### Configure **AWS CodeBuild to Build the Docker Image**

1. **Create a Build Project:**
    - Give the project a name (for example, **`cloudmartBuild`**).
    - Connect it to your existing GitHub repository (**`cloudmart-application`**).
    - **Image: amazonlinux2-x86_64-standard:4.0**
    - Configure the environment to support Docker builds. Enable "Enable this flag if you want to build Docker images or want your builds to get elevated privileges"
    - Add the environment variable **ECR_REPO** with the ECR repository URI.
    - For the build specification, use the following **`buildspec.yml`**:
  
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

  
   - ![Image8](https://github.com/user-attachments/assets/7dab9252-5735-4709-aac1-c88b2626dc40) - ![Image9](https://github.com/user-attachments/assets/81de8a8f-a44d-4ea1-ba2f-e78073888bfd)
  - ![Image10](https://github.com/user-attachments/assets/b1989099-0646-40b2-999d-ee49c9f79827)

 **Add the AmazonElasticContainerRegistryPublicFullAccess permission to ECR in the service role**
- Access the IAM console > Roles.
- Look for the role created "cloudmartBuild" for CodeBuild.
- Add the permission **AmazonElasticContainerRegistryPublicFullAccess**.

- ![Image11](https://github.com/user-attachments/assets/c21b5070-66c6-4f0f-a51c-b531b85c6f11)
- ![Image12](https://github.com/user-attachments/assets/edd23b87-b7e2-4029-acd4-9f40c1af2f51)
- ![Image13](https://github.com/user-attachments/assets/6c8c6fb4-ba17-46ef-ac09-a9f77d4dfb42)



































    

