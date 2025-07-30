pipeline {
    agent any

    environment {
        IMAGE_NAME = "rajneshyadav/nodejs-app"
        REGISTRY = "docker.io"
        GIT_REPO = "https://github.com/RajneshKumarDevops/DevSecOps-pipeline.git"
        GITOPS_REPO = "git@github.com:RajneshKumarDevops/GitOps.git"

        VAULT_SECRET_GITHUB = 'secret/github'
        VAULT_SECRET_DOCKERHUB = 'secret/dockerhub'
        VAULT_SECRET_SONAR = 'secret/sonarqube'
        VAULT_SECRET_GITOPS = 'secret/gitops'
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    try {
                        withVault([vaultSecrets: [[path: "${VAULT_SECRET_GITHUB}", secretValues: [[envVar: 'GITHUB_TOKEN', vaultKey: 'token']]]]]) {
                            checkout scm: [
                                $class: 'GitSCM',
                                branches: [[name: '*/main']],
                                userRemoteConfigs: [[
                                    url: "${GIT_REPO}",
                                    credentials: [username: 'RajneshKumarDevops', password: "${GITHUB_TOKEN}"]
                                ]]
                            ]
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Checkout failed: ${e.message}"
                    }
                }
            }
        }

        stage('Prepare') {
            steps {
                script {
                    try {
                        env.IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Failed to determine image tag: ${e.message}"
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    try {
                        withVault([vaultSecrets: [[path: "${VAULT_SECRET_SONAR}", secretValues: [[envVar: 'SONAR_TOKEN', vaultKey: 'token']]]]]) {
                            def scannerHome = tool name: 'SonarQube Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                            withSonarQubeEnv('SonarQube') {
                                sh """
                                    ${scannerHome}/bin/sonar-scanner \
                                    -Dsonar.projectKey=nodejs-app \
                                    -Dsonar.sources=. \
                                    -Dsonar.exclusions=**/*.java \
                                    -Dsonar.login=$SONAR_TOKEN
                                """
                            }
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "SonarQube analysis failed: ${e.message}"
                    }
                }
            }
        }

        stage('Dependency Scan with Grype') {
            steps {
                sh "grype . -o table > grype-deps-report.txt || true"
                sh 'cat grype-deps-report.txt'
            }
        }

        stage('Security Scan with Trivy (FS)') {
            steps {
                sh 'trivy fs --scanners vuln --no-progress --severity HIGH,CRITICAL --format table --output trivy-fs-report.txt . || true'
                sh 'cat trivy-fs-report.txt'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Docker build failed: ${e.message}"
                    }
                }
            }
        }

        stage('Scan Docker Image') {
            steps {
                script {
                    try {
                        sh """
                            trivy image --timeout 10m \
                            --scanners vuln \
                            --no-progress \
                            --severity HIGH,CRITICAL \
                            --format table \
                            --output trivy-report.txt \
                            ${IMAGE_NAME}:${IMAGE_TAG} || true
                        """
                        sh 'cat trivy-report.txt'
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Docker image scan failed: ${e.message}"
                    }
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    try {
                        withVault([vaultSecrets: [[path: "${VAULT_SECRET_DOCKERHUB}", secretValues: [
                            [envVar: 'DOCKER_USER', vaultKey: 'username'],
                            [envVar: 'DOCKER_PASS', vaultKey: 'password']
                        ]]]]) {
                            sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                            sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Docker push failed: ${e.message}"
                    }
                }
            }
        }

        stage('GitOps Update') {
            steps {
                script {
                    try {
                        withVault([vaultSecrets: [[path: "${VAULT_SECRET_GITOPS}", secretValues: [[envVar: 'SSH_KEY', vaultKey: 'key']]]]]) {
                            sh 'rm -rf temp-repo'
                            sh "mkdir -p ~/.ssh && echo \"$SSH_KEY\" > ~/.ssh/id_rsa && chmod 600 ~/.ssh/id_rsa"
                            sh "ssh-keyscan github.com >> ~/.ssh/known_hosts"
                            sh "git clone ${GITOPS_REPO} temp-repo"
                            dir('temp-repo') {
                                sh "sed -i 's|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yaml"
                                script {
                                    def changes = sh(script: "git status --porcelain", returnStdout: true).trim()
                                    if (changes) {
                                        sh "git add ."
                                        sh "git commit -m 'Update image tag to ${IMAGE_TAG}'"
                                        sh "git push origin main"
                                    } else {
                                        echo "No changes detected in GitOps repo."
                                    }
                                }
                            }
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "GitOps update failed: ${e.message}"
                    }
                }
            }
        }

        stage('Sync ArgoCD') {
            steps {
                script {
                    try {
                        sh "argocd app sync nodejs-app"
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "ArgoCD sync failed: ${e.message}"
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                echo "Build Status: ${currentBuild.currentResult}"
                archiveArtifacts artifacts: '*.txt', fingerprint: true
            }
        }
    }
}
