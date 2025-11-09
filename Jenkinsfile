@Library('Shared') _ 
pipeline {
    agent any
    
    environment {
        SONAR_HOME = tool "Sonar"
    }
    
    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
    }

    stages {

        stage("Validate Parameters") {
            steps {
                script {
                    if (params.FRONTEND_DOCKER_TAG == '' || params.BACKEND_DOCKER_TAG == '') {
                        error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }

        stage("Workspace cleanup") {
            steps {
                script {
                    cleanWs()
                    sh 'echo WORKSPACE CLEANUP SUCCESS'
                    sh 'echo ====================='
                }
            }
        }
        
        stage('Git: Code Checkout') {
            steps {
                script {
                    checkOut("https://github.com/mashoodkhan/3-tier-secops.git", "main")
                    sh 'echo CODE CHECKOUT SUCCESS'
                    sh 'echo ============================================================'
                }
            }
        }
        
        stage("Trivy: Filesystem scan") {
            steps {
                script {
                    trivy_scan()
                    sh 'echo FILE SYSTEM SCAN SUCCESSFUL!'
                    sh 'echo ====================='
                }
            }
        }

        stage("SonarQube: Code Analysis") {
            steps {
                script {
                    sonarqube_analysis("Sonar","3-tier-secops","3-tier-secops")
                    sh "echo SONAR CODE ANALYSIS PASSED!"
                    sh 'echo ====================='
                }
            }
        }
        
        stage('Exporting environment variables') {
            parallel {
                stage("Backend env setup") {
                    steps {
                        script {
                            dir("Automations") {
                                sh "bash updatebackendnew.sh"
                                sh "echo UPDATED BACKEND ENVs"
                                sh 'echo ====================='
                            }
                        }
                    }
                }
                
                stage("Frontend env setup") {
                    steps {
                        script {
                            dir("Automations") {
                                sh "bash updatefrontendnew.sh"
                                sh "echo UPDATED FRONTEND ENVs"
                                sh 'echo ====================='
                            }
                        }
                    }
                }
            }
        }
        
        stage("Docker: Build & Push Images to Docker Hub") {
            steps {
                script {
                    buildAndPushDocker("3-tier-secops", "${params.BACKEND_DOCKER_TAG}", "mashoodk", "backend")
                    buildAndPushDocker("3-tier-secops", "${params.FRONTEND_DOCKER_TAG}", "mashoodk", "frontend")
                }
            }
        }

        stage("Docker: Build & Push Images to ECR") {
            parallel {
                stage("Backend Image") {
                    steps {
                        script {
                            buildAndPushImageToECR("3-tier-secops", "${params.BACKEND_DOCKER_TAG}", "321262206944", "us-east-2", "backend")
                        }
                    }
                }
                stage("Frontend Image") {  
                    steps {
                        script {
                            buildAndPushImageToECR("3-tier-secops", "${params.FRONTEND_DOCKER_TAG}", "321262206944", "us-east-2", "frontend")
                        }
                    }
                }
            }
        }   //  closes ECR stage

    } //  closes stages block

} //  closes pipeline block
