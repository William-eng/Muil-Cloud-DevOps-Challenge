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


































































































      
