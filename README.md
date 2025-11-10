# devops-ci-cd-pipeline
CI/CD pipeline including build, test, code quality, packaging, containerization, and deployment to a staging environment.

---

## Overview
This project demonstrates a complete DevOps CI/CD pipeline for a Spring Boot application using **Jenkins**, **Maven**, **SonarQube**, **Nexus**, **Docker**, and **Docker Compose**. The pipeline automates:

- Code checkout from GitHub
- Unit testing and build
- Code quality analysis with SonarQube
- Artifact deployment to Nexus
- Docker image build and push
- Deployment to a staging environment using Docker Compose

The pipeline is configured to run on multiple VMs:

- **VM1:** Jenkins (with Java, Maven, and Docker installed)
- **VM2:** SonarQube and Nexus (Dockerized)
- **VM3:** Staging environment (Docker and Docker Compose)

---

## CI/CD Pipeline Steps

1. **Clone GitHub Repository**  
   Pulls the latest code from the main branch using SSH credentials.

2. **Maven Test**  
   Runs unit tests.  
   Tools used: Maven 3.9.11 and JDK 17.

3. **SonarQube Analysis**  
   Performs static code analysis to maintain code quality.

4. **Maven Package**  
   Packages the application into a deployable JAR file.

5. **Deploy Artifact to Nexus** *(manual confirmation required)*  
   Uploads the packaged artifact to a Nexus snapshot repository for version control.

6. **Docker Login**  
   Authenticates with Docker Hub to enable image build and push.

7. **Build and Push Docker Image** *(manual confirmation required)*  
   Builds a Docker image of the Spring Boot application and pushes it to Docker Hub.

8. **Deploy to Staging Environment** *(manual confirmation required)*  
   Connects using SHH, Copies the docker-compose.yml to the staging VM and runs Docker Compose to pull the latest image and restart the application.

---

## Pipeline Structure and Configuration

- **Jenkinsfile:** Defines all pipeline stages, environment variables, and credentials.  
- **Dockerfile:** Builds the application Docker image.  
- **docker-compose.yml:** Defines the deployment configuration on the staging environment.

**Triggering:**  
The pipeline can be triggered automatically on changes to the main branch (github webhook enabled) or manually via Jenkins.

---

## Credentials & Tools

- **Jenkins Credentials:**
  - GitHub SSH key for repo checkout
  - Docker Hub login for image push
  - Staging VM SSH key for deployment
  - SonarQube token for code analysis
  - Nexus credentials for artifact deployment

- **Tools installed on Jenkins VM:**
  - Java 17 (Added to System tools)
  - Maven 3.9.11 (Added to System tools)
  - Docker
    
 - **Installed Plugins:**
  - Git, GitHub
  - Docker Pipeline
  - SSH Agent Plugin
  - SonarQube Scanner
  - Config File Provider
    
 - **Jenkins Global Environment Variables :**
  - STAGING_VM: user@public-ip
  - NEXUS_SNAPSHOT_URL: http://nexus-server/repository/maven-snapshots/
    
 - **Config File Management:**
  - maven-settings: Content -> settings.xml file

---

## References
- For detailed logs, screenshots, and architecture diagrams, please refer to the [docs folder](docs/).

---

