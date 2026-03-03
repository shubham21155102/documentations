# 🔧 Jenkins

> Frequently used Jenkins concepts, CLI commands, and pipeline patterns.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [Installation](#installation)
- [Jenkins CLI](#jenkins-cli)
- [Jenkinsfile Declarative Pipeline](#jenkinsfile-declarative-pipeline)
- [Scripted Pipeline](#scripted-pipeline)
- [Shared Libraries](#shared-libraries)
- [Docker in Jenkins](#docker-in-jenkins)
- [Credentials & Secrets](#credentials--secrets)
- [Useful Groovy Snippets](#useful-groovy-snippets)

---

## Installation

```bash
# Install Jenkins on Ubuntu
sudo apt update
sudo apt install -y openjdk-17-jdk
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt update && sudo apt install -y jenkins

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# Default port: http://localhost:8080
```

**Docker-based Jenkins:**

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
```

---

## Jenkins CLI

```bash
# Download Jenkins CLI jar
curl -O http://localhost:8080/jnlpJars/jenkins-cli.jar

# Run command
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth user:token help

# List jobs
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth user:token list-jobs

# Build a job
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth user:token build myjob

# Build with parameters
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth user:token build myjob -p ENV=production

# Get build output
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth user:token console myjob

# Enable / disable job
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth user:token enable-job myjob
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth user:token disable-job myjob

# Delete job
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth user:token delete-job myjob

# Reload configuration
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth user:token reload-configuration

# Restart Jenkins
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth user:token restart

# Safe restart (wait for builds to finish)
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth user:token safe-restart
```

---

## Jenkinsfile Declarative Pipeline

```groovy
pipeline {
    agent any  // or agent { label 'linux' }

    environment {
        APP_NAME = 'myapp'
        REGISTRY = 'myrepo'
    }

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to build')
        booleanParam(name: 'DEPLOY', defaultValue: false, description: 'Deploy after build?')
        choice(name: 'ENV', choices: ['dev', 'staging', 'production'], description: 'Target environment')
    }

    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    triggers {
        pollSCM('H/5 * * * *')       // poll every 5 minutes
        cron('0 2 * * *')             // nightly build
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'
                    }
                    post {
                        always {
                            junit 'target/surefire-reports/*.xml'
                        }
                    }
                }
                stage('Lint') {
                    steps {
                        sh 'mvn checkstyle:check'
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    def image = docker.build("${REGISTRY}/${APP_NAME}:${BUILD_NUMBER}")
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {
                        image.push()
                        image.push('latest')
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                expression { params.DEPLOY == true }
            }
            steps {
                sh """
                    kubectl set image deployment/${APP_NAME} \
                        app=${REGISTRY}/${APP_NAME}:${BUILD_NUMBER}
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            slackSend(color: 'good', message: "Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
        failure {
            slackSend(color: 'danger', message: "Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
            mail to: 'team@example.com', subject: "FAILED: ${env.JOB_NAME}", body: "Build URL: ${env.BUILD_URL}"
        }
    }
}
```

---

## Scripted Pipeline

```groovy
node('linux') {
    try {
        stage('Checkout') {
            checkout scm
        }

        stage('Build') {
            sh 'make build'
        }

        stage('Test') {
            sh 'make test'
        }

        currentBuild.result = 'SUCCESS'
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        stage('Cleanup') {
            cleanWs()
        }
    }
}
```

---

## Shared Libraries

**Directory structure (`vars/deployApp.groovy`):**

```groovy
// vars/deployApp.groovy
def call(String appName, String environment) {
    sh "helm upgrade --install ${appName} ./charts/${appName} -n ${environment}"
}
```

**Usage in Jenkinsfile:**

```groovy
@Library('my-shared-library') _

pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                deployApp('myapp', 'staging')
            }
        }
    }
}
```

---

## Docker in Jenkins

```groovy
pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            args '-v $HOME/.npm:/root/.npm'
        }
    }
    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}
```

---

## Credentials & Secrets

```groovy
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                // Username/password
                withCredentials([usernamePassword(
                    credentialsId: 'db-credentials',
                    usernameVariable: 'DB_USER',
                    passwordVariable: 'DB_PASS'
                )]) {
                    sh 'psql -U $DB_USER -d mydb'
                }

                // Secret text
                withCredentials([string(credentialsId: 'api-key', variable: 'API_KEY')]) {
                    sh 'curl -H "Authorization: Bearer $API_KEY" https://api.example.com'
                }

                // SSH key
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ssh-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {
                    sh 'ssh -i $SSH_KEY user@server.example.com'
                }

                // File
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh 'kubectl get pods'
                }
            }
        }
    }
}
```

---

## Useful Groovy Snippets

```groovy
// Get git commit hash
def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()

// Get branch name
def branch = env.GIT_BRANCH.replaceAll('origin/', '')

// Conditional based on branch
if (env.BRANCH_NAME == 'main') {
    // do something
}

// Read a file
def content = readFile('config.json')

// Write a file
writeFile file: 'output.txt', text: 'Hello World'

// Archive artifacts
archiveArtifacts artifacts: 'target/*.jar', fingerprint: true

// Stash / unstash files between stages
stash includes: 'target/*.jar', name: 'build-artifacts'
unstash 'build-artifacts'

// Set build description
currentBuild.description = "Version: ${VERSION}"

// Mark build as unstable
currentBuild.result = 'UNSTABLE'

// Input approval
input message: 'Deploy to production?', ok: 'Deploy'
```

---

[← Back to Home](../README.md)
