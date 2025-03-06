# DAY TWO: MultiCloud, DevOps & AI Challenge

### Scenerio
Redesign and migrate application to run on modern cloud Environment with DevOps Integration and AI support to reduce Customer's cost 

## Part 1 - Docker

### Step 1: Install Docker on EC2
Execute the following commands:

      sudo yum update -y
      sudo yum install docker -y
      sudo systemctl start docker
      sudo docker run hello-world
      sudo systemctl enable docker
      docker --version
      sudo usermod -a -G docker $(whoami)
      newgrp docker

      sudo usermod -a -G docker $(whoami)
      newgrp docker

- ![Image1](https://github.com/user-attachments/assets/d4098354-9277-4abe-800b-165dbfe737b8)
  
## Step 2: Create Docker image for CloudMart

### Backend

Create folder and download source code:

      mkdir -p challenge-day2/backend && cd challenge-day2/backend
      wget https://tcb-public-events.s3.amazonaws.com/mdac/resources/day2/cloudmart-backend.zip
      unzip cloudmart-backend.zip
- ![Image2](https://github.com/user-attachments/assets/357ea3bb-0306-42fe-b2dc-044c9c685d0a)

Create .env file:

      nano .env

Content of .env:

      PORT=5000
      AWS_REGION=us-east-1
      BEDROCK_AGENT_ID=<your-bedrock-agent-id>
      BEDROCK_AGENT_ALIAS_ID=<your-bedrock-agent-alias-id>
      OPENAI_API_KEY=<your-openai-api-key>
      OPENAI_ASSISTANT_ID=<your-openai-assistant-id>

Create Dockerfile:

      nano Dockerfile
      
Content of Dockerfile:

      FROM node:18
      WORKDIR /usr/src/app
      COPY package*.json ./
      RUN npm install
      COPY . .
      EXPOSE 5000
      CMD ["npm", "start"]
- ![Image3](https://github.com/user-attachments/assets/70792add-07f3-45b2-81fb-8e2e9b4a75d7)


### Frontend

Create folder and download source code:

      cd ..
      mkdir frontend && cd frontend
      wget https://tcb-public-events.s3.amazonaws.com/mdac/resources/day2/cloudmart-frontend.zip
      unzip cloudmart-frontend.zip

- ![Image4](https://github.com/user-attachments/assets/b6b43d5e-fddb-4a74-9dfb-594ee657893a)

Create Dockerfile:

      nano Dockerfile

Content of Dockerfile:
This is a Multi-stage Docker file to reduces the final image size by using intermediate stages to build the application, ensuring only necessary artifacts are included in the final image. This improves security, performance, and deployment efficiency by eliminating unnecessary dependencies and files.

            FROM node:16-alpine as build
            WORKDIR /app
            COPY package*.json ./
            RUN npm ci
            COPY . .
            RUN npm run build
            
            FROM node:16-alpine
            WORKDIR /app
            RUN npm install -g serve
            COPY --from=build /app/dist /app
            ENV PORT=5001
            ENV NODE_ENV=production
            EXPOSE 5001
            CMD ["serve", "-s", ".", "-l", "5001"]



- ![Image5](https://github.com/user-attachments/assets/c3e7609f-6559-4077-a05a-ea2b4e94f02a)

## Part 2 - Kubernetes

Kubernetes efficiently manages containerized applications, and using a multistage Dockerfile helps optimize deployment by reducing image size and improving security. **Attention:** AWS Kubernetes service (EKS) is not free, so you will incur charges while executing the hands-on tasks—remember to delete the cluster afterward using the removal section to avoid unnecessary costs.

## Cluster Setup on AWS Elastic Kubernetes Services (EKS)

- Create a user named `eksuser` with Admin privileges and authenticate with it

        aws configure

            [ec2-user@ip-172-31-87-172 frontend]$ aws configure
            AWS Access Key ID [None]: <Your-Key-ID>
            AWS Secret Access Key [None]: <Your-Access-Key>
            Default region name [None]: us-east-1
            Default output format [None]: json


- Install the CLI tool eksctl

        curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
        sudo cp /tmp/eksctl /usr/bin
        eksctl version

- ![Image6](https://github.com/user-attachments/assets/216f5542-6207-4045-8a48-c0c8b00d2af9)


- Install the CLI tool kubectl

      curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.18.9/2020-11-02/bin/linux/amd64/kubectl
      chmod +x ./kubectl
      mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
      echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
      kubectl version --short --client


- ![Image7](https://github.com/user-attachments/assets/0a08c531-3b46-45fd-8faa-0e53056826be)


- Create an EKS Cluster

                    eksctl create cluster \
                          --name cloudmart \
                          --region us-east-1 \
                          --nodegroup-name standard-workers \
                          --node-type t3.medium \
                          --nodes 1 \
                          --with-oidc \
                          --managed


- ![Image8](https://github.com/user-attachments/assets/05e6399f-6443-4afb-9381-9c04fcfba178)
- ![Image9](https://github.com/user-attachments/assets/e62394f9-262f-4350-b8c6-198624ab00b7)



- Connect to the EKS cluster using the kubectl configuration

              aws eks update-kubeconfig --name cloudmart


- Verify Cluster Connectivity

              kubectl get svc
              kubectl get nodes


- Create a Role & Service Account to provide pods access to services used by the application (DynamoDB, Bedrock, etc).


            eksctl create iamserviceaccount \
              --cluster=cloudmart \
              --name=cloudmart-pod-execution-role \
              --role-name CloudMartPodExecutionRole \
              --attach-policy-arn=arn:aws:iam::aws:policy/AdministratorAccess\
              --region us-east-1 \
              --approve


- ![Image10](https://github.com/user-attachments/assets/ca6d7ce0-4316-4ca3-8eab-a128942f00a0)

**NOTE**: In the example above, Admin privileges were used to facilitate educational purposes. Always remember to follow the principle of least privilege in production environments

## Backend Deployment on Kubernetes

### **Create an ECR Repository for the Backend and upload the Docker image to it**

- ![Image11](https://github.com/user-attachments/assets/30e159ef-fde2-4b5f-ae93-ae1d6081c019)
- ![Image12](https://github.com/user-attachments/assets/d08742c9-9f4d-41b4-8500-ac86e05023dd)



### Switch to backend folder

            cd ../..
            cd challenge-day2/backend


### Follow the ECR steps to build your Docker image

- ![Image13](https://github.com/user-attachments/assets/b88ba639-bff8-414f-8e83-e9c6fb579777)



            cd ../..
            cd challenge-day2/backend
            nano cloudmart-backend.yaml

- ![Image14](https://github.com/user-attachments/assets/aeba9ee9-fc17-4f43-9652-7164f7b9d4f5)

- ![Image15](https://github.com/user-attachments/assets/f536cb10-1fd3-445d-9032-20b165077c31)

### **Create a Kubernetes deployment file (YAML) for the Backend**

            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: cloudmart-backend-app
            spec:
              replicas: 1
              selector:
                matchLabels:
                  app: cloudmart-backend-app
              template:
                metadata:
                  labels:
                    app: cloudmart-backend-app
                spec:
                  serviceAccountName: cloudmart-pod-execution-role
                  containers:
                  - name: cloudmart-backend-app
                    image: public.ecr.aws/<your-id>/cloudmart-backend:latest
                    env:
                    - name: PORT
                      value: "5000"
                    - name: AWS_REGION
                      value: "us-east-1"
                    - name: BEDROCK_AGENT_ID
                      value: "xxxxxx"
                    - name: BEDROCK_AGENT_ALIAS_ID
                      value: "xxxx"
                    - name: OPENAI_API_KEY
                      value: "xxxxxx"
                    - name: OPENAI_ASSISTANT_ID
                      value: "xxxx"
            ---
            
            apiVersion: v1
            kind: Service
            metadata:
              name: cloudmart-backend-app-service
            spec:
              type: LoadBalancer
              selector:
                app: cloudmart-backend-app
              ports:
                - protocol: TCP
                  port: 5000
                  targetPort: 5000


### Deploy the Backend on Kubernetes

            kubectl apply -f cloudmart-backend.yaml

Monitor the status of objects being created and obtain the public IP generated for the API




- ![Image16](https://github.com/user-attachments/assets/ddf4157e-6420-4df4-a5c1-1abcb1bceb70)


## Frontend Deployment on Kubernetes

### Preparation

Change the Frontend's .env file to point to the API URL created within Kubernetes obtained by the `kubectl get service` command

- ![Image17](https://github.com/user-attachments/assets/114e822e-440b-4256-9aa1-818a3bc44306)

### Create an ECR Repository for the Frontend and upload the Docker image to it

Repository name: cloudmart-frontend

- ![Image18](https://github.com/user-attachments/assets/b0553c0a-c566-46cf-9759-74831660ee25)

### Follow the ECR steps to build your Docker image

- ![Image19](https://github.com/user-attachments/assets/f813d8ec-7e18-4427-9b96-2e73dbfdacbe)
- ![Image20](https://github.com/user-attachments/assets/2f4622b1-03da-45bc-bc57-ea3a545f0139)


### **Create a Kubernetes deployment file (YAML) for the Frontend**

            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: cloudmart-frontend-app
            spec:
              replicas: 1
              selector:
                matchLabels:
                  app: cloudmart-frontend-app
              template:
                metadata:
                  labels:
                    app: cloudmart-frontend-app
                spec:
                  serviceAccountName: cloudmart-pod-execution-role
                  containers:
                  - name: cloudmart-frontend-app
                    image: public.ecr.aws/l4c0j8h9/cloudmart-frontend:latest
            ---
            
            apiVersion: v1
            kind: Service
            metadata:
              name: cloudmart-frontend-app-service
            spec:
              type: LoadBalancer
              selector:
                app: cloudmart-frontend-app
              ports:
                - protocol: TCP
                  port: 5001
                  targetPort: 5001

- ![Image21](https://github.com/user-attachments/assets/1f7ecdd6-7be6-4576-b7ad-0a508d8863c9)

  Now Let check the Loadbalancer on the AWS Page and access our application

  - ![Image22](https://github.com/user-attachments/assets/c87522fe-b4a2-4267-bba2-54ade0f3501d)
- ![Image23](https://github.com/user-attachments/assets/b703caa4-f33c-4e42-a4bc-9e0a928e8fe3)

  We can now access our application on port 5000 of the frontend  loadbancer 

- ![Image24](https://github.com/user-attachments/assets/adc595eb-0b3a-435e-9a4f-4eec19747976)
- ![Image25](https://github.com/user-attachments/assets/ca52fa4c-c70d-48ce-a718-751f83f26212)
- ![Image26](https://github.com/user-attachments/assets/2e86cfa2-89f0-4549-afbf-f70423e9887e)
- ![Image27](https://github.com/user-attachments/assets/81465726-a10d-43b9-a07e-ec6664e3b452)



### Removal
At the end of the hands-on, delete all resources:
If you delete the cluster at the end of the exercise, you'll have to recreate it for the next days. So decide what makes more sense for you: delete the cluster and recreate it every day or keep it and pay for the time it's running. However, don't forget to delete it permanently at the end of the Challenge.


            kubectl delete service cloudmart-frontend-app-service
            kubectl delete deployment cloudmart-frontend-app
            kubectl delete service cloudmart-backend-app-service
            kubectl delete deployment cloudmart-backend-app
            
            eksctl delete cluster --name cloudmart --region us-east-1








































      
