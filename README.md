Production-Ready Node.js Microservice on AWS EC2

Project Overview

This project demonstrates how to deploy a containerized Node.js microservice to AWS EC2 using Docker, Terraform, GitHub Actions, Docker Hub, CloudWatch, Prometheus, and Grafana.

The project is designed for beginners who are new to:

AWS

Node.js

Docker

Terraform

GitHub

GitHub Actions

CI/CD

Prometheus

Grafana

The implementation starts with a basic Node.js application and gradually adds containerization, infrastructure as code, automated deployment, and monitoring.

Final Architecture

Developer Laptop
       |
       v
GitHub Repository
       |
       v
GitHub Actions CI/CD
       |
       +--> Build Docker Image
       |
       +--> Push Image to Docker Hub
       |
       +--> SSH into AWS EC2
       |
       +--> Pull Latest Docker Image
       |
       +--> Restart Node.js Container
       |
       v
AWS EC2 Instance
       |
       +--> Node.js App     :3000
       +--> Prometheus      :9090
       +--> Grafana         :3001
       |
       v
AWS CloudWatch Monitoring

Technology Stack

Component

Purpose

Node.js

Backend microservice

Express.js

Web framework

Docker

Application containerization

Docker Hub

Container image registry

GitHub

Source-code repository

GitHub Actions

CI/CD pipeline

Terraform

AWS infrastructure provisioning

AWS EC2

Application hosting

AWS CloudWatch

Infrastructure monitoring

Prometheus

Application metrics collection

Grafana

Metrics visualization

Phase 1 — Prerequisites

Create the following accounts:

AWS account

GitHub account

Docker Hub account

Install the following software on the local Windows computer:

Visual Studio Code

Git

Node.js LTS

Docker Desktop

Terraform

AWS CLI

Verify the installations:

git --version
node -v
npm -v
docker --version
terraform --version
aws --version

Phase 2 — Create the Node.js Project

Step 1 — Create the project folder

Open PowerShell or the VS Code terminal:

cd C:\Users\krish
mkdir nodejs-microservice
cd nodejs-microservice
code .

The terminal path should now be similar to:

C:\Users\krish\nodejs-microservice

Step 2 — Initialize the Node.js application

npm init -y

This creates:

package.json

Step 3 — Install Express

npm install express

Step 4 — Create index.js

Create the following file in the root project folder:

nodejs-microservice/index.js

Add:

const express = require('express');

const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.send('Node.js Microservice Running');
});

app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'healthy'
  });
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});

Step 5 — Update package.json

Replace the scripts section with:

"scripts": {
  "start": "node index.js"
}

Step 6 — Test locally

npm start

Open:

http://localhost:3000

Health endpoint:

http://localhost:3000/health

Stop the local app using:

CTRL + C

Phase 3 — Dockerize the Application

Step 1 — Create the Dockerfile

Create this file in the root project folder:

nodejs-microservice/Dockerfile

The filename must be exactly:

Dockerfile

Do not use:

DockerFile
Dockerfile.txt
dockerfile

Add:

FROM node:18

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]

Step 2 — Create .dockerignore

Create:

nodejs-microservice/.dockerignore

Add:

node_modules
npm-debug.log
.git
terraform

Step 3 — Build the Docker image

Run this command from the root project folder:

docker build -t nodejs-microservice .

Step 4 — Verify the image

docker images

Step 5 — Run the container

docker run -d --name node-app-local -p 3000:3000 nodejs-microservice

Verify:

docker ps

Open:

http://localhost:3000

Port 3000 already in use

Check the process:

netstat -ano | findstr :3000

Stop the process using its PID:

taskkill /PID <PID_NUMBER> /F

Or stop an existing Docker container:

docker ps
docker stop <CONTAINER_ID>
docker rm <CONTAINER_ID>

Phase 4 — Push the Image to Docker Hub

Step 1 — Create a Docker Hub repository

Create a public repository named:

nodejs-microservice

Step 2 — Login from the terminal

docker login

Step 3 — Tag the image

Replace YOUR_DOCKER_USERNAME:

docker tag nodejs-microservice YOUR_DOCKER_USERNAME/nodejs-microservice:latest

Step 4 — Push the image

docker push YOUR_DOCKER_USERNAME/nodejs-microservice:latest

Verify the image in Docker Hub.

Phase 5 — Configure AWS CLI

Create an IAM user for Terraform and development access.

For a learning lab, attach the required AWS permissions. Avoid using broad administrator permissions in production.

Configure AWS CLI:

aws configure

Enter:

AWS Access Key ID:     <your-access-key>
AWS Secret Access Key:<your-secret-key>
Default region name:  ap-south-1
Default output format: json

Verify:

aws sts get-caller-identity

Never upload AWS access keys to GitHub.

Phase 6 — Create AWS Infrastructure with Terraform

Recommended folder structure

nodejs-microservice/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── terraform/
│   ├── main.tf
│   └── modules/
│       ├── networking/
│       │   └── main.tf
│       └── ec2/
│           └── main.tf
├── .dockerignore
├── .gitignore
├── Dockerfile
├── index.js
├── package.json
└── package-lock.json

Step 1 — Create SSH key

On Windows PowerShell:

ssh-keygen -t ed25519

Press Enter to accept the default path.

The key files are normally created at:

C:\Users\krish\.ssh\id_ed25519
C:\Users\krish\.ssh\id_ed25519.pub

The .pub file is the public key.

The file without .pub is the private key and must remain secret.

Step 2 — Networking module

Create:

terraform/modules/networking/main.tf

Add:

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "nodejs-vpc"
  }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-south-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "nodejs-public-subnet"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "nodejs-igw"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "nodejs-public-route-table"
  }
}

resource "aws_route" "internet_access" {
  route_table_id         = aws_route_table.public.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.igw.id
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

output "vpc_id" {
  value = aws_vpc.main.id
}

output "subnet_id" {
  value = aws_subnet.public.id
}

Step 3 — EC2 module

Create:

terraform/modules/ec2/main.tf

Add:

variable "vpc_id" {
  type = string
}

variable "subnet_id" {
  type = string
}

resource "aws_security_group" "ec2_sg" {
  name        = "nodejs-sg"
  description = "Security group for Node.js monitoring lab"
  vpc_id      = var.vpc_id

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Node.js App"
    from_port   = 3000
    to_port     = 3000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Grafana"
    from_port   = 3001
    to_port     = 3001
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Prometheus"
    from_port   = 9090
    to_port     = 9090
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "nodejs-sg"
  }
}

resource "aws_key_pair" "deployer" {
  key_name   = "nodejs-key"
  public_key = file("C:/Users/krish/.ssh/id_ed25519.pub")
}

data "aws_ami" "ubuntu" {
  most_recent = true

  owners = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

resource "aws_instance" "app_server" {
  ami                         = data.aws_ami.ubuntu.id
  instance_type               = "t2.micro"
  subnet_id                   = var.subnet_id
  vpc_security_group_ids      = [aws_security_group.ec2_sg.id]
  key_name                    = aws_key_pair.deployer.key_name
  associate_public_ip_address = true

  tags = {
    Name = "Nodejs-Microservice"
  }
}

output "public_ip" {
  value = aws_instance.app_server.public_ip
}

output "instance_id" {
  value = aws_instance.app_server.id
}

Step 4 — Root Terraform file

Create:

terraform/main.tf

Add:

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}

module "networking" {
  source = "./modules/networking"
}

module "ec2" {
  source    = "./modules/ec2"
  vpc_id    = module.networking.vpc_id
  subnet_id = module.networking.subnet_id
}

output "public_ip" {
  value = module.ec2.public_ip
}

output "instance_id" {
  value = module.ec2.instance_id
}

Step 5 — Initialize and deploy

Move into the Terraform root directory:

cd C:\Users\krish\nodejs-microservice\terraform

Run:

terraform init
terraform fmt -recursive
terraform validate
terraform plan
terraform apply

Type:

yes

Copy the public IP from the Terraform output.

Phase 7 — Install Docker on EC2

SSH from PowerShell:

ssh -i C:/Users/krish/.ssh/id_ed25519 ubuntu@EC2_PUBLIC_IP

Install Docker:

sudo apt update
sudo apt install docker.io -y
sudo systemctl enable --now docker
sudo usermod -aG docker ubuntu

Exit and reconnect:

exit

Reconnect and verify:

docker --version

Phase 8 — GitHub Actions CI/CD

Step 1 — Create the workflow location

From the project root:

mkdir .github
mkdir .github\workflows

Create:

.github/workflows/deploy.yml

Step 2 — Add workflow

name: Deploy Node.js App

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker Image
        run: |
          docker build \
            -t ${{ secrets.DOCKER_USERNAME }}/nodejs-microservice:${{ github.sha }} \
            -t ${{ secrets.DOCKER_USERNAME }}/nodejs-microservice:latest \
            .

      - name: Push Docker Image
        run: |
          docker push ${{ secrets.DOCKER_USERNAME }}/nodejs-microservice:${{ github.sha }}
          docker push ${{ secrets.DOCKER_USERNAME }}/nodejs-microservice:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            set -e

            docker pull ${{ secrets.DOCKER_USERNAME }}/nodejs-microservice:${{ github.sha }}

            docker rm -f node-app || true

            docker run -d \
              --name node-app \
              --restart unless-stopped \
              -p 3000:3000 \
              ${{ secrets.DOCKER_USERNAME }}/nodejs-microservice:${{ github.sha }}

            sleep 5
            docker ps
            curl --fail http://localhost:3000/health

Step 3 — Add GitHub repository secrets

Open:

Repository
→ Settings
→ Secrets and variables
→ Actions

Add:

DOCKER_USERNAME
DOCKER_PASSWORD
EC2_HOST
EC2_SSH_KEY

EC2_HOST must contain only the EC2 public or Elastic IP.

EC2_SSH_KEY must contain the complete private key:

-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----

Step 4 — Create .gitignore

Create:

.gitignore

Add:

node_modules/
terraform/.terraform/
terraform/*.tfstate
terraform/*.tfstate.*
*.pem
.env

Keep .terraform.lock.hcl in Git because it locks the provider version.

Step 5 — Initialize and push Git

Run these commands from the project root:

git init
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_GITHUB_EMAIL"
git add .
git commit -m "Initial CI/CD setup"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/nodejs-microservice.git
git push -u origin main

If origin already exists:

git remote -v
git remote set-url origin https://github.com/YOUR_USERNAME/nodejs-microservice.git

Phase 9 — Add Prometheus Metrics to Node.js

Step 1 — Install prom-client locally

Run this on the local Windows computer, not inside EC2:

cd C:\Users\krish\nodejs-microservice
npm install prom-client

If PowerShell blocks npm scripts:

Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

Close and reopen VS Code, then retry:

npm install prom-client

Step 2 — Update index.js

Replace index.js with:

const express = require('express');
const client = require('prom-client');

const app = express();
const PORT = 3000;

client.collectDefaultMetrics();

app.get('/', (req, res) => {
  res.send('Node.js Microservice Running');
});

app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'healthy'
  });
});

app.get('/metrics', async (req, res) => {
  try {
    res.set('Content-Type', client.register.contentType);
    res.end(await client.register.metrics());
  } catch (error) {
    res.status(500).end(error.message);
  }
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});

Step 3 — Commit and deploy

git add package.json package-lock.json index.js
git commit -m "Add Prometheus metrics endpoint"
git push

Wait for GitHub Actions to finish successfully.

Verify:

http://EC2_PUBLIC_IP:3000
http://EC2_PUBLIC_IP:3000/health
http://EC2_PUBLIC_IP:3000/metrics

Phase 10 — Configure Prometheus

Step 1 — Create a monitoring network

SSH into EC2:

ssh -i C:/Users/krish/.ssh/id_ed25519 ubuntu@EC2_PUBLIC_IP

Run:

docker network create monitoring || true
docker network connect monitoring node-app || true

Step 2 — Create Prometheus configuration

mkdir -p ~/monitoring
cd ~/monitoring
nano prometheus.yml

Add:

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "nodejs-app"
    metrics_path: /metrics
    static_configs:
      - targets:
          - "node-app:3000"

  - job_name: "prometheus"
    static_configs:
      - targets:
          - "localhost:9090"

Save with:

CTRL + O
Enter
CTRL + X

Step 3 — Validate the Prometheus configuration

docker run --rm \
  -v "$PWD/prometheus.yml:/etc/prometheus/prometheus.yml:ro" \
  prom/prometheus \
  promtool check config /etc/prometheus/prometheus.yml

Expected:

SUCCESS

Step 4 — Start Prometheus

docker rm -f prometheus 2>/dev/null || true

docker run -d \
  --name prometheus \
  --restart unless-stopped \
  --network monitoring \
  -p 9090:9090 \
  -v "$PWD/prometheus.yml:/etc/prometheus/prometheus.yml:ro" \
  -v prometheus-data:/prometheus \
  prom/prometheus

Step 5 — Verify Prometheus

docker ps
docker logs --tail 50 prometheus
curl http://localhost:9090/-/healthy

Expected:

Prometheus Server is Healthy.

Open:

http://EC2_PUBLIC_IP:9090

Open the targets page:

http://EC2_PUBLIC_IP:9090/targets

Expected states:

nodejs-app    UP
prometheus    UP

Test a query:

process_resident_memory_bytes

Phase 11 — Configure Grafana

Step 1 — Start Grafana

docker rm -f grafana 2>/dev/null || true

docker run -d \
  --name grafana \
  --restart unless-stopped \
  --network monitoring \
  -p 3001:3000 \
  -v grafana-data:/var/lib/grafana \
  grafana/grafana

Verify:

docker ps
docker logs --tail 50 grafana

Open:

http://EC2_PUBLIC_IP:3001

Default login:

Username: admin
Password: admin

Change the password when prompted.

Step 2 — Add Prometheus as a data source

In Grafana:

Connections
→ Data sources
→ Add data source
→ Prometheus

Use this URL:

http://prometheus:9090

Do not use the public IP when Grafana and Prometheus are on the same Docker network.

Click:

Save & Test

Expected:

Successfully queried the Prometheus API

Phase 12 — Create Grafana Panels

CPU panel

Create a new dashboard and add a visualization.

Use the query:

rate(process_cpu_user_seconds_total{job="nodejs-app"}[5m])

Panel settings:

Visualization: Time series
Title: Node.js CPU Usage
Time range: Last 15 minutes
Refresh: 5 seconds

Memory panel

Use:

process_resident_memory_bytes{job="nodejs-app"}

Panel settings:

Visualization: Time series
Title: Node.js Resident Memory
Unit: bytes (IEC)

Application availability panel

Use:

up{job="nodejs-app"}

Recommended visualization:

Stat

Interpretation:

1 = UP
0 = DOWN

Generate activity

Refresh the application several times:

http://EC2_PUBLIC_IP:3000
http://EC2_PUBLIC_IP:3000/health
http://EC2_PUBLIC_IP:3000/metrics

Wait for at least two Prometheus scrape intervals before checking the graph.

Phase 13 — Troubleshoot Grafana “No Data”

Use this order.

Check 1 — Node.js metrics endpoint

curl http://localhost:3000/metrics

Expected output includes:

process_cpu_user_seconds_total
process_resident_memory_bytes

Check 2 — Prometheus target status

Open:

http://EC2_PUBLIC_IP:9090/targets

The nodejs-app target must show:

UP

Check 3 — Query Prometheus directly

Open Prometheus and execute:

process_resident_memory_bytes{job="nodejs-app"}

If Prometheus returns no data, Grafana cannot display data.

Check 4 — Confirm Docker network

docker network inspect monitoring

The output should include:

node-app
prometheus
grafana

Connect missing containers:

docker network connect monitoring node-app
docker network connect monitoring prometheus
docker network connect monitoring grafana

Check 5 — Verify Grafana data source URL

Use:

http://prometheus:9090

Check 6 — Run the Grafana query

In the panel editor:

Select Prometheus data source

Select Time series

Enter the PromQL query

Click Run queries

Set time range to Last 15 minutes

Set refresh to 5s

Save the panel

Phase 14 — Configure CloudWatch

EC2 basic metrics are automatically available in CloudWatch.

CPU alarm

Open:

AWS Console
→ CloudWatch
→ Alarms
→ Create alarm
→ Select metric
→ EC2
→ Per-Instance Metrics
→ CPUUtilization

Configure:

Threshold type: Static
Condition: Greater than
Threshold: 70
Period: 5 minutes

Alarm name:

Nodejs-EC2-High-CPU

Status-check alarm

Select:

StatusCheckFailed

Configure:

Condition: Greater than or equal to 1
Period: 1 minute

Alarm name:

Nodejs-EC2-Status-Check-Failed

SNS notifications may be added later for email alerts.

Phase 15 — Assign an Elastic IP

A normal EC2 public IP changes after stopping and starting the instance.

To keep a stable address:

AWS Console
→ EC2
→ Network & Security
→ Elastic IPs
→ Allocate Elastic IP address
→ Allocate

Select the new Elastic IP:

Actions
→ Associate Elastic IP address
→ Select the EC2 instance
→ Associate

Update the GitHub secret:

EC2_HOST

with the Elastic IP.

After this, use:

http://ELASTIC_IP:3000
http://ELASTIC_IP:3000/metrics
http://ELASTIC_IP:9090
http://ELASTIC_IP:3001

Phase 16 — Container Auto-Restart

All production containers should use:

--restart unless-stopped

Verify:

docker inspect -f '{{.Name}} -> {{.HostConfig.RestartPolicy.Name}}' \
  node-app prometheus grafana

Expected:

/node-app -> unless-stopped
/prometheus -> unless-stopped
/grafana -> unless-stopped

Enable Docker at boot:

sudo systemctl enable docker
sudo systemctl start docker

Verification Checklist

EC2 containers

docker ps

Expected:

node-app
prometheus
grafana

Node.js

http://ELASTIC_IP:3000

Health check

http://ELASTIC_IP:3000/health

Metrics

http://ELASTIC_IP:3000/metrics

Prometheus

http://ELASTIC_IP:9090

Prometheus targets

http://ELASTIC_IP:9090/targets

Grafana

http://ELASTIC_IP:3001

GitHub Actions

Expected:

build  ✅
deploy ✅

Common Errors and Fixes

Dockerfile not found

Error:

failed to read dockerfile

Fix:

Rename DockerFile to Dockerfile

Port 3000 already allocated

docker ps
docker rm -f <container-using-port-3000>

SSH private key not found in GitHub Actions

Verify EC2_SSH_KEY contains the complete private key, not the .pub file.

Terraform public key format error

Terraform must use:

id_ed25519.pub

not:

id_ed25519

Nested Git repository error

Error:

terraform/ does not have a commit checked out

Remove the nested Git metadata from PowerShell:

Remove-Item -Recurse -Force terraform\.git

PowerShell blocks npm

Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

Grafana shows no data

Check in this order:

Node.js /metrics
→ Prometheus target UP
→ Prometheus query returns data
→ Grafana data source test succeeds
→ Grafana PromQL query runs

Security Limitations in the Initial Version

The first version intentionally prioritizes learning and visibility over complete production security.

Known limitations:

HTTP instead of HTTPS

Publicly exposed monitoring ports

SSH initially open to broad source ranges

Secrets stored in GitHub repository secrets

No load balancer

No auto scaling

No blue-green deployment

No SAST, DAST, SCA, or secret scanning

No centralized log collection

Shift-Left Security Improvements

Recommended improvements:

Add HTTPS using an Application Load Balancer or NGINX

Restrict SSH to the administrator’s trusted IP

Replace direct SSH with AWS Systems Manager Session Manager

Store application secrets in AWS Secrets Manager

Add SAST using Semgrep or SonarQube

Add SCA and container scanning using Trivy

Add secret scanning using Gitleaks

Add pre-commit hooks

Add Jest tests and an 80% coverage threshold

Add a health check after deployment

Add blue-green deployment

Add CloudWatch log collection

Place Grafana and Prometheus behind controlled access

Add an Application Load Balancer and Auto Scaling Group

Cleanup to Avoid AWS Charges

Destroy Terraform-managed infrastructure:

cd terraform
terraform destroy

Type:

yes

Release the Elastic IP after disassociating it if it is no longer required.

Confirm that the following resources are removed:

EC2 instance

Elastic IP

VPC

Subnet

Internet Gateway

Security group

Route table

Project Outcome

This project demonstrates:

Node.js microservice development

Docker image creation

Docker Hub image management

Modular Terraform

AWS VPC and EC2 provisioning

GitHub Actions CI/CD

Automated EC2 deployment

Prometheus metrics collection

Grafana dashboard creation

CloudWatch alarms

Elastic IP management

Container restart policies

Troubleshooting of real deployment issues

Author

Created as a hands-on DevOps, cloud, CI/CD, and monitoring learning project.
