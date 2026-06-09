Got it — you want the raw text to paste directly into that GitHub editor. Here it is, just select all and replace what's there:

---

```
# CI-CD-GitOps — FastAPI on AWS EKS with Jenkins & Argo CD

A end-to-end CI/CD pipeline that builds a containerised FastAPI application, scans it for vulnerabilities, pushes it to Amazon ECR, and deploys it to an EKS cluster via GitOps (Argo CD).

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Project Structure](#project-structure)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Pipeline Stages](#pipeline-stages)
- [Configuration](#configuration)
- [Application](#application)
- [Known Issues & Notes](#known-issues--notes)

---

## Architecture Overview

```
Developer Push
      │
      ▼
GitHub (Source Repo)
      │
      ▼
Jenkins Pipeline
  ├── Checkout
  ├── Set Image Tag  (BUILD_NUMBER-<git-short-hash>)
  ├── SonarQube Analysis
  ├── Docker Build
  ├── Trivy Image Scan
  ├── Push to Amazon ECR
  └── Update GitOps Repo (deployment.yaml)
                │
                ▼
         GitHub (GitOps Repo)
                │
                ▼
           Argo CD
                │
                ▼
        AWS EKS Cluster
```

---

## Project Structure

```
.
├── main.py                  # FastAPI application entry point
├── form.html                # Landing page template
├── output.html              # Deployment output / CI dashboard template
├── requirements.txt         # Python dependencies
├── Dockerfile               # Container image definition
├── Jenkinsfile              # Declarative Jenkins pipeline
└── sonar-project.properties # SonarQube project configuration
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Application | Python 3.9, FastAPI, Uvicorn, Jinja2 |
| Containerisation | Docker |
| CI Server | Jenkins |
| Code Quality | SonarQube + sonar-scanner |
| Image Security | Trivy |
| Container Registry | Amazon ECR |
| GitOps / CD | Argo CD |
| Orchestration | Amazon EKS (Kubernetes) |

---

## Prerequisites

Before running the pipeline you need:

- A running **Jenkins** instance with the following plugins: Git, Pipeline, SonarQube Scanner, Amazon ECR, AWS Credentials
- **Docker** installed on the Jenkins agent
- **Trivy** installed on the Jenkins agent (`trivy` on `$PATH`)
- **sonar-scanner** installed at `/opt/sonar-scanner/bin/`
- A configured **SonarQube** server
- An **Amazon ECR** repository
- An **EKS** cluster with Argo CD installed and pointed at your GitOps repo
- A separate **GitOps repository** containing `k8s/deployment.yaml`

### Jenkins Credentials Required

| Credential ID | Type | Used For |
|---|---|---|
| `git-cred` | Username / Password | GitHub checkout & GitOps repo push |
| `sonar-cred` | Secret Text | SonarQube token |
| `aws-cred` | AWS Credentials | ECR login & push |

---

## Getting Started

### 1. Clone this repository

```bash
git clone https://github.com/<your-username>/CI-CD-Gitops.git
cd CI-CD-Gitops
```

### 2. Run the application locally

```bash
pip install -r requirements.txt
uvicorn main:app --reload
```

The app will be available at `http://127.0.0.1:8000`.

### 3. Build and run with Docker

```bash
docker build -t lwm-app .
docker run -p 80:80 lwm-app
```

### 4. Configure the Jenkinsfile

Open `Jenkinsfile` and update the environment variables marked with `#change as per your ...`:

```groovy
environment {
    SONARQUBE_SERVER = 'SonarQubeServer'
    IMAGE_NAME       = 'alphaimage'
    ECR_REPO         = '<account-id>.dkr.ecr.<region>.amazonaws.com/<repo-name>'
    AWS_REGION       = 'ap-south-1'
    DEPLOYMENT_REPO  = 'https://github.com/<your-username>/CI-CD-dest-repo.git'
}
```

### 5. Create a Jenkins pipeline job

- New Item → Pipeline → point SCM to this repository
- Jenkins will automatically detect and run the `Jenkinsfile`

---

## Pipeline Stages

| Stage | Description |
|---|---|
| **Checkout** | Clones the source repository from GitHub |
| **Set Image Tag** | Generates a unique tag: `<BUILD_NUMBER>-<git-short-hash>` |
| **SonarQube Analysis** | Static code analysis using sonar-scanner |
| **Build Docker Image** | Builds the image and tags it with both the unique tag and `latest` |
| **Image Scan** | Runs a Trivy vulnerability scan; results saved to `report.txt` |
| **Push to ECR** | Authenticates with AWS and pushes both tags to ECR |
| **Update GitOps Repo** | Clones the GitOps repo, updates `k8s/deployment.yaml` with the new image tag, and commits & pushes |

---

## Configuration

### sonar-project.properties

```properties
sonar.projectKey=lwm-fastapi-project
sonar.projectName=lwm FastAPI Project
sonar.projectVersion=1.0
sonar.sources=.
sonar.language=python
sonar.sourceEncoding=UTF-8
```

Update `sonar.projectKey` and `sonar.projectName` to match your SonarQube project.

### Dockerfile

The image is based on `python:3.9`. The app runs on port `80` via Uvicorn. Templates (`form.html`, `output.html`) are copied to `/code` and served by Jinja2.

---

## Application

The FastAPI app exposes two routes:

| Route | Description |
|---|---|
| `GET /` | Renders the welcome page (`form.html`) |
| `GET /output` | Renders the deployment output dashboard (`output.html`) |

The `output.html` template is a Jinja2-powered CI dashboard that displays build logs, deployment metadata (environment, cluster, namespace, replicas), and quick actions (Retry Deploy, Open Argo CD).

---

## Known Issues & Notes

- The `/output` route in `main.py` is indented inside the `/` handler and will never be registered. Fix:

```python
@app.get("/")
def form_post(request: Request):
    return templates.TemplateResponse('form.html', context={'request': request})

@app.get("/output")
def output_page(request: Request):
    return templates.TemplateResponse('output.html', context={'request': request})
```

- The Trivy scan writes `report.txt` but the pipeline does not fail on critical vulnerabilities. Add a threshold check if needed.
- Ensure Argo CD is configured with auto-sync so deployments roll out automatically after the image tag is updated.

---

## Contributing

Pull requests are welcome. For major changes please open an issue first to discuss what you would like to change.

## License

This project is for educational purposes as part of the **Learn With Mithran** series by **Greens Technologies**.
```

---

Just **select all** the text in the GitHub editor, **paste** this in, then hit **Commit changes**. The Preview tab will show you the rendered version before you commit.
