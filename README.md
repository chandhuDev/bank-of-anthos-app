# Bank of Anthos Application with Jenkins CI/CD

This repository contains the source code for the Bank of Anthos microservices application along with Jenkins pipeline configurations for automated building and pushing of Docker images.

## Project Overview

A microservices-based banking application featuring:
- Automated CI/CD using Jenkins
- Containerization with Docker
- Multiple service components
- Integration with [Bank of Anthos Kubernetes Deployment](https://github.com/chandhuDev/bank-of-anthos-k8)

## Microservices

1. **Frontend Service**
   - Python/Flask web interface
   - Handles user interactions
   - Dockerfile: `Dockerfile/frontend/Dockerfile`

2. **Account Services**
   - `userservice`: User authentication & management
   - `contacts`: Contact information management
   - Dockerfiles in respective service directories

3. **Transaction Services**
   - `balancereader`: Account balance queries
   - `ledgerwriter`: Transaction processing
   - `transactionhistory`: Historical transaction data
   - Individual Dockerfiles for each service

## CI/CD Pipeline

### Jenkins Pipeline Stages
1. Checkout
   - Fetches source code
   - Validates repository structure

2. Build
   - Compiles each service
   - Runs unit tests
   - Generates artifacts

3. Docker Image Build
   - Creates optimized containers
   - Tags images appropriately
   - Implements multi-stage builds

4. Push to Registry
   - Authenticates with Docker Hub
   - Pushes tagged images
   - Updates latest tags

### Image Repositories
All images are pushed to Docker Hub under:
- `chandhudev0/boa-frontend`
- `chandhudev0/boa-userservice`
- `chandhudev0/boa-contacts`
- `chandhudev0/boa-ledgerwriter`
- `chandhudev0/boa-balancereader`
- `chandhudev0/boa-transactionhistory`

## Setup and Usage

1. Jenkins Configuration
   ```bash
   # Configure Jenkins credentials:
   - Docker Hub credentials
   - GitHub repository access
   - Build triggers
2. Local Devlopment
  # Clone repository
    git clone https://github.com/chandhuDev/bank-of-anthos-app.git

  # Build individual services
    cd src/<service-name>
    docker build -t <service-name> -f Dockerfile/<service-name>/Dockerfile .
3. Pipeline Execution
   - Triggered on push to master
   - Manual execution available
   - Parameterized builds supported

     
Integration with Kubernetes Deployment
This repository works in conjunction with Bank of Anthos K8s Deployment:

 - CI/CD pipeline builds images
 - Images pushed to Docker Hub
 - ArgoCD detects new images
 - Automatic deployment to Kubernetes cluster

Monitoring and Logging

Build Monitoring

 - Jenkins build status
 - Pipeline stage views
 - Build logs and artifacts

Image Status

 - Docker Hub repositories
 - Image tags and versions
 - Pull statistics



Troubleshooting
Common Issues:

Build Failures

 - Check service dependencies
 - Verify Docker daemon status
 - Review Jenkins logs


Push Failures
  - Verify Docker Hub credentials
  - Check network connectivity
  - Review image tags
