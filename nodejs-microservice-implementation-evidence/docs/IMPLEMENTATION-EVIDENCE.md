# Implementation Evidence

Screenshots captured from the actual local/AWS implementation of this Node.js Microservice project.

## Repository Structure

```text
nodejs-microservice/
├── README.md
├── screenshots/
│   ├── 01-monitoring/
│   ├── 02-aws-infrastructure/
│   ├── 03-ci-cd/
│   ├── 04-docker/
│   ├── 05-terraform/
│   ├── 06-aws-cli-iam/
│   ├── 07-application/
│   └── 08-soc-lab/
└── docs/
    └── IMPLEMENTATION-EVIDENCE.md
```

## 1. Application Monitoring

### Grafana Dashboard
![Grafana](../screenshots/01-monitoring/Grafana.png)

### Prometheus Targets
![Prometheus](../screenshots/01-monitoring/prometheus.png)

### Node.js Metrics
![Node.js Metrics](../screenshots/01-monitoring/nodejs-metrics.png)

### AWS CloudWatch
![CloudWatch](../screenshots/01-monitoring/cloudwatch.png)

## 2. AWS Infrastructure

### EC2 Running
![EC2](../screenshots/02-aws-infrastructure/ec2-running.png)

### EC2 Deployment
![EC2 Deployment](../screenshots/02-aws-infrastructure/ec2-deployment.png)

## 3. CI/CD

### GitHub Actions Successful Deployment
![CI/CD](../screenshots/03-ci-cd/ci-cd-successful.png)

### GitHub Actions Build
![GitHub Actions](../screenshots/03-ci-cd/github-actions-build.png)

## 4. Docker

### Dockerfile and Build
![Dockerfile](../screenshots/04-docker/dockerfile-and-build.png)

### Docker Login
![Docker Login](../screenshots/04-docker/docker-login.png)

### Docker Image Pushed to Docker Hub
![Docker Hub](../screenshots/04-docker/docker-image-pushed-to-dockerhub.png)

### Docker Container
![Docker Container](../screenshots/04-docker/docker-container-created.png)

### Docker Pulled and Running on EC2
![Docker on EC2](../screenshots/04-docker/docker-pulled-and-running-on-ec2.png)

## 5. Terraform

### Terraform Initialization
![Terraform Init](../screenshots/05-terraform/terraform-initialized.png)

### Terraform Validation
![Terraform Validate](../screenshots/05-terraform/terraform-validated.png)

## 6. AWS CLI and IAM

### AWS CLI Connected with IAM
![AWS CLI](../screenshots/06-aws-cli-iam/aws-cli-iam-connection.png)

### IAM User
![IAM User](../screenshots/06-aws-cli-iam/iam-user-created.png)

## 7. Node.js Application

### Node.js Microservice Running
![Node.js](../screenshots/07-application/nodejs-running.png)

## 8. SOC Lab

### Wazuh SOC Dashboard
![Wazuh](../screenshots/08-soc-lab/wazuh-soc-dashboard.png)

### AWS SOC Lab Topology
![SOC Lab](../screenshots/08-soc-lab/soc-lab-aws-topology.jpeg)

> These screenshots are implementation evidence from the author's local development and AWS lab environments. IP addresses and environment-specific values may differ when reproducing the project.
