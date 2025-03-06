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















































































































      
