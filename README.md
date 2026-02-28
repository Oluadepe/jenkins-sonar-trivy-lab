# Jenkins + SonarQube + Trivy Practice Repo

A hands-on repo to learn Jenkins pipelines by building, scanning, and enforcing gates.

## What you will learn
- How Jenkins finds and runs a Jenkinsfile from your repo
- How SonarQube scan + Quality Gate blocks bad code
- How Trivy scans a Docker image and fails on HIGH/CRITICAL vulnerabilities

## Quick start (SonarQube local)
```bash
docker compose -f compose.sonarqube.yml up -d
```

Open SonarQube: http://localhost:9000  
Default login: admin / admin (change password on first login)

Create a token:
- My Account → Security → Generate Token

## Jenkins setup (high level)
1) Install plugin: **SonarQube Scanner for Jenkins**  
2) Manage Jenkins → System → SonarQube servers:
   - Name: `SonarQube-Server`
   - URL: `http://<sonarqube-host>:9000`
   - Credentials: Sonar token (Secret text)
3) Manage Jenkins → Tools:
   - Add SonarQube Scanner (auto-install)
4) Create a Pipeline job:
   - Definition: Pipeline script from SCM
   - Script Path: `Jenkinsfile`

## Run
Click **Build Now** in Jenkins and watch stages:
Checkout → Test → SonarQube Scan → Quality Gate → Docker Build → Trivy Scan
