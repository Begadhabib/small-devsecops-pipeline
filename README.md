# small-devsecops-pipeline

![Architecture Diagram](https://github.com/user-attachments/assets/400bd0ba-ce53-4b17-b1a5-3a5e0a36bb4d)

## Project Overview
This project is designed as a hands-on DevSecOps pipeline for a Node.js chess web app. The goal is to demonstrate how to automate code quality, security, and deployment using modern CI/CD practices. It’s a practical example for learning, experimenting, and showing best practices in secure software delivery.

## Why This Setup?
- **Security First:** Integrates static code analysis (SonarQube), open source dependency scanning (OWASP Dependency-Check), and container vulnerability scanning (Trivy) to catch bugs and vulnerabilities early.
- **Automation:** Jenkins automates every step, reducing human error and speeding up delivery.
- **Clean Builds:** Each pipeline run starts from a clean workspace, ensuring no leftover files or secrets.
- **Cloud Native:** Uses Docker and Kubernetes for scalable, reproducible deployments.

## Jenkins Pipeline Breakdown
The Jenkinsfile orchestrates the following steps:

1. **Clean Workspace**
   - Uses `cleanWs()` to remove all files from previous builds. This is important for security and consistency.
2. **Checkout Source Code**
   - Pulls the latest code from GitHub, ensuring the pipeline always works with the newest changes.
3. **Install Dependencies**
   - Runs `npm install` inside the Chess/ directory to fetch all Node.js dependencies.
4. **SonarQube Analysis**
   - Uses SonarQube to scan the code for bugs, code smells, and security vulnerabilities (SAST).
   - The scanner is configured via Jenkins tools and environment variables.
5. **Quality Gate**
   - Waits for SonarQube’s quality gate. If the code doesn’t meet the required standards, the pipeline fails and stops further steps.
6. **OWASP Dependency-Check**
   - Scans Node.js dependencies for known vulnerabilities using OWASP Dependency-Check and publishes a report.
7. **Trivy Filesystem Scan**
   - Runs Trivy to scan the project filesystem for vulnerabilities before building the Docker image.
8. **Docker Build**
   - Builds and tags the Docker image for the app.
9. **Trivy Docker Image Scan**
   - Scans the built Docker image for vulnerabilities using Trivy and archives the report.
10. **Docker Push**
    - Pushes the image to Docker Hub (or your registry).
11. **Deploy to Container**
    - Runs the Docker container locally for quick validation.
12. **Deploy to Kubernetes**
    - Applies all manifests in the `k8s/` folder and shows the status in the `chess` namespace.

> **Security Tools Used:**
> - **SonarQube:** Static code analysis for security and code quality.
> - **OWASP Dependency-Check:** Scans open source dependencies for vulnerabilities.
> - **Trivy:** Scans both the filesystem and Docker images for vulnerabilities.
> - **Jenkins:** Automates and enforces the pipeline logic.
> - **Git:** Source control for traceability and collaboration.
> - **cleanWs:** Prevents data leaks and ensures clean builds.

## App Structure
- All source code is in the `Chess/` folder
- `server.js` runs the backend and serves the UI
- Frontend: HTML/CSS/JS with real-time play via Socket.io

## Docker & Kubernetes
- `Dockerfile` for building the app image
- `k8s/Deployment.yaml`: Deploys the app (4 replicas, resource limits, environment variables)
- `k8s/service.yaml`: Exposes the app via NodePort (port 30000) for easy local access
- `k8s/namespace.yaml`: Isolates resources in the `chess` namespace

## How to Run

### Local
```bash
cd Chess
npm install
npm start
```
Visit [http://localhost:3000](http://localhost:3000)

### Docker
```bash
docker build -t my_chess_app:latest ./Chess
docker run -p 3000:3000 my_chess_app:latest
```

### Kubernetes
```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/Deployment.yaml
kubectl apply -f k8s/service.yaml
```
Access at: `http://<your-node-ip>:30000`

## Jenkins Setup
- Requires Node.js, JDK, SonarQube, Trivy, and OWASP Dependency-Check tools configured in Jenkins
- Connect Jenkins to your GitHub repo
- Configure SonarQube server in Jenkins
- Add Docker Hub and Kubernetes credentials as needed
- Run the pipeline and watch the stages in Jenkins



---

