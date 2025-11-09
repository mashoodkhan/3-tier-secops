@Library('Shared') _
pipeline {
    agent any
    
    environment {
        SONAR_HOME = tool "Sonar"
        AWS_ACCOUNT_ID = "321262206944"
        AWS_REGION = "us-east-2"
        ECR_REPO = "3-tier-secops"
    }
    
    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '', description: 'Frontend Docker tag')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: '', description: 'Backend Docker tag')
    }

    stages {

        stage("Validate Parameters") {
            steps {
                script {
                    if (!params.FRONTEND_DOCKER_TAG || !params.BACKEND_DOCKER_TAG) {
                        error("Both FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }

        stage("Workspace cleanup") {
            steps { cleanWs() }
        }
        
        stage('Git: Code Checkout') {
            steps {
                script {
                    checkOut("https://github.com/mashoodkhan/3-tier-secops.git", "main")
                }
            }
        }

        stage("Trivy: Filesystem scan") {
            steps { script { trivy_scan() } }
        }

        stage("SonarQube: Code Analysis") {
            steps { script { sonarqube_analysis("Sonar","3-tier-secops","3-tier-secops") } }
        }

        stage('Exporting environment variables') {
            parallel {
                stage("Backend env setup") {
                    steps { dir("Automations") { sh "bash updatebackendnew.sh" } }
                }
                stage("Frontend env setup") {
                    steps { dir("Automations") { sh "bash updatefrontendnew.sh" } }
                }
            }
        }

        stage("Docker: Build & Push Images to Docker Hub") {
            steps {
                script {
                    buildAndPushDocker(ECR_REPO, params.BACKEND_DOCKER_TAG, "mashoodk", "backend")
                    buildAndPushDocker(ECR_REPO, params.FRONTEND_DOCKER_TAG, "mashoodk", "frontend")
                }
            }
        }

        stage("Create ECR Repo (One-Time)") {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: AWS_REGION) {
                        sh """
                            aws ecr describe-repositories --repository-names ${ECR_REPO} \
                            || aws ecr create-repository --repository-name ${ECR_REPO} --region ${AWS_REGION}
                        """
                    }
                }
            }
        }

        stage("Docker: Build & Push Images to ECR") {
            parallel {
                stage("Backend Image") {
                    steps {
                        script {
                            buildAndPushImageToECR(ECR_REPO, params.BACKEND_DOCKER_TAG, AWS_ACCOUNT_ID, AWS_REGION, "backend")
                        }
                    }
                }
                stage("Frontend Image") {
                    steps {
                        script {
                            buildAndPushImageToECR(ECR_REPO, params.FRONTEND_DOCKER_TAG, AWS_ACCOUNT_ID, AWS_REGION, "frontend")
                        }
                    }
                }
            }
        }
    }
}
