# 📝 API Automation Test with Postman & Newman

## 📋 Overview

This project API testing using **Postman** and **Newman**.
It includes automated execution, JSON Schema validation, HTML reporting, and **CI/CD pipeline with Jenkins**.

---

## ⚙️ Prerequisites

- Node.js ≥ v14
- npm
- Postman
- Newman
- Docker (for Jenkins pipeline)

Install globally:

```bash
npm install -g newman newman-reporter-htmlextra
```

Or install locally (recommended):

```bash
npm install newman newman-reporter-htmlextra
```

---

## 📁 Project Structure

```
API-automation-project-folder/
├── API_Example_Collection.postman_collection.json   # Postman test collection
├── API_Example_Env.postman_environment.json         # Postman environment variables
├── API_Test_Report.html                             # Generated HTML test report
├── Jenkinsfile                                      # Jenkins pipeline definition
├── JenkinsDockerfile                                # Custom Jenkins Docker image
└── README.md
```

---

## 🚀 Running Tests Locally

```bash
newman run API_Example_Collection.postman_collection.json \
  -e API_Example_Env.postman_environment.json \
  -r cli,html,junit \
  --reporter-html-export API_Test_Report.html \
  --reporter-junit-export results.xml
```

---

## 🐳 Jenkins CI/CD Pipeline

### Architecture

This project uses a **custom Jenkins Docker image** that has Node.js and Newman pre-installed, allowing the pipeline to run API tests automatically on every commit.

```
GitHub Repo → Jenkins (Docker) → Newman runs tests → HTML Report archived
```

### JenkinsDockerfile

The custom Jenkins image is built from `JenkinsDockerfile`:

```dockerfile
FROM jenkins/jenkins:lts

USER root

RUN apt-get update && \
    apt-get install -y docker.io curl && \
    curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
    apt-get install -y nodejs && \
    npm install -g newman newman-reporter-html

USER jenkins
```

### Build & Run Jenkins

**Step 1:** Build the custom Jenkins image

```bash
docker build -t my-jenkins -f JenkinsDockerfile .
```

**Step 2:** Run Jenkins container

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e JAVA_OPTS="-Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400" \
  my-jenkins
```

> **Note:** The `JAVA_OPTS` flag fixes a known issue (`JENKINS-48300`) where Jenkins on macOS with Docker volumes reports `exit code -1` due to slow filesystem heartbeat detection.

**Step 3:** Access Jenkins at [http://localhost:8080](http://localhost:8080)

### Jenkinsfile

The pipeline is defined in `Jenkinsfile`:

```groovy
pipeline {
    agent any

    parameters {
        choice(
            name: 'ENV_FILE',
            choices: ['API_Example_Env.postman_environment.json'],
            description: 'Select Postman Environment'
        )
    }

    stages {
        stage('Run API Test') {
            steps {
                sh """
                newman run API_Example_Collection.postman_collection.json \
                    -e ${params.ENV_FILE} \
                    -r cli,html,junit \
                    --reporter-html-export API_Test_Report.html \
                    --reporter-junit-export results.xml
                """
            }
        }

        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: 'API_Test_Report.html'
                junit 'results.xml'
            }
        }
    }
}
```

### Pipeline Stages

| Stage | Description |
|-------|-------------|
| **Run API Test** | Executes Newman with the selected environment file, generates HTML and JUnit reports |
| **Archive Report** | Archives `API_Test_Report.html` as a build artifact and publishes JUnit test results |

### Pipeline Result Interpretation

| Status | Meaning |
|--------|---------|
| ✅ **SUCCESS** | All API tests passed |
| 🟡 **UNSTABLE** | Tests ran but some assertions failed |
| ❌ **FAILURE** | Pipeline error or critical test failure |

> **Tip:** If the pipeline shows `FAILURE` but Newman ran successfully, check if any test assertions are failing (exit code 1). Review the Console Output for `AssertionError` details.

---

## 🔧 Troubleshooting

### `exit code -1` on macOS
Jenkins on macOS with Docker volumes may show this error due to filesystem latency. Fix by adding this to your `docker run` command:
```bash
-e JAVA_OPTS="-Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
```

### Newman not found in Jenkins
Verify that Jenkins is running the correct custom image (not the official `jenkins/jenkins:lts`):
```bash
docker ps --format "table {{.Names}}\t{{.Image}}"
docker exec jenkins which newman
```

### Assertion Errors in Test Results
If you see `AssertionError` in the console (e.g., `data.createdAt should be string`), the API response format doesn't match the expected JSON schema in Postman. Update the test assertion in Postman to match the actual API response type.
