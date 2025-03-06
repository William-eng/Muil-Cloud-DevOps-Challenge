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














































































      
