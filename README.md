Maven CICD Pipeline
A complete end-to-end CI/CD pipeline for a Spring Boot application using Jenkins, SonarQube, Docker, ArgoCD, and Kubernetes.
Architecture
Developer pushes code to GitHub
         ↓
      Jenkins (CI)
         ↓
    SonarQube (Code Quality)
         ↓
    DockerHub (Image Registry)
         ↓
      ArgoCD (CD)
         ↓
   Kubernetes (Deployment)
Tech Stack
ToolPurposeJenkinsCI/CD OrchestrationSonarQubeStatic Code AnalysisDockerContainerizationDockerHubImage RegistryArgoCDGitOps Continuous DeliveryKubernetes/MinikubeContainer OrchestrationMavenBuild ToolSpring BootApplication Framework
Prerequisites

AWS EC2 instance (for Jenkins and SonarQube)
Minikube (for local Kubernetes)
DockerHub account
GitHub account

Infrastructure Setup
Jenkins EC2

Ubuntu 24.04
Jenkins installed and running on port 8080
Docker installed and configured
Java 17 installed

SonarQube EC2

Ubuntu 24.04
SonarQube 10.x running on port 9000
Java 17 installed

Jenkins Pipeline Stages
1. Checkout
Pulls latest code from GitHub repository.
2. Build and Test
bashmvn clean package
Compiles the Java code and runs unit tests using Maven.
3. Static Code Analysis
bashmvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}
```
Sends code to SonarQube for quality analysis. Pipeline stops if quality gate fails.

### 4. Build and Push Docker Image
Builds multi-architecture Docker image (AMD64 + ARM64) using QEMU and pushes to DockerHub with build number as tag.

### 5. Update Deployment File
Automatically updates `deployment.yml` in GitHub with the new image tag. ArgoCD detects this change and deploys automatically.

## Jenkins Credentials Required

| Credential ID | Type | Description |
|--------------|------|-------------|
| `sonarqube` | Secret text | SonarQube authentication token |
| `docker-cred` | Username/Password | DockerHub credentials |
| `github` | Secret text | GitHub personal access token |

## Docker Image

Multi-architecture image supporting both AMD64 and ARM64:
```
sathwik1reddy/ultimate-cicd:BUILD_NUMBER
    ├── AMD64 image
    └── ARM64 image
ArgoCD Setup
ArgoCD is installed directly on Minikube:
bashkubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
Access ArgoCD UI:
bashkubectl port-forward svc/argocd-server -n argocd 8080:443
Get admin password:
bashkubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

## GitHub Webhook Setup

For fully automated pipeline triggering:

1. Go to GitHub repo → **Settings** → **Webhooks**
2. Add webhook:
   - Payload URL: `http://JENKINS_EC2_IP:8080/github-webhook/`
   - Content type: `application/json`
   - Events: Push events only
3. In Jenkins job → **Build Triggers** → enable **GitHub hook trigger for GITScm polling**

## How It Works
```
1. Developer pushes code to GitHub
2. GitHub webhook triggers Jenkins automatically
3. Jenkins builds and tests the code
4. SonarQube analyzes code quality
5. Docker image built for AMD64 + ARM64
6. Image pushed to DockerHub with build number tag
7. deployment.yml updated with new image tag
8. ArgoCD detects change in GitHub
9. ArgoCD deploys new image to Kubernetes
10. Application is live!
Rollback
ArgoCD supports one-click rollback to any previous version:

Go to ArgoCD UI
Click History and Rollback
Select the version to rollback to
Click Rollback

Lessons Learned

SonarQube 10.x requires Java 17
Apple Silicon (ARM64) requires multi-arch Docker images
QEMU enables building ARM64 images on AMD64 machines
ArgoCD operator not compatible with ARM64 — use direct install
GitHub webhooks eliminate manual pipeline triggering

Author
Sathwik Reddy
