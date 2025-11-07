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

    stages { // <--- wrap all stages inside this block

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

        // stage("OWASP: Dependency check"){
        //     steps{
        //         script{
        //             owasp_dependency()
        //         }
        //     }
        // }
        
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarqube_analysis("Sonar","3-tier-secops","3-tier-secops")
                     sh "echo SONAR CODE ANAYLYSIS PASSED!"
                    sh 'echo ====================='
                }
            }
        }
        
        // stage("SonarQube: Code Quality Gates"){
        //     steps{
        //         script{
        //             sonarqube_code_quality()
        //              sh "echo SONAR QUALITY GATES PASSED!"
        //             sh 'echo ====================='
        //         }
        //     }
        // }
        
        stage('Exporting environment variables') {
            parallel{
                stage("Backend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatebackendnew.sh"
                                sh "echo UPDATED BACKEND ENVs"
                                sh 'echo ====================='
                            }
                        }
                    }
                }
                
                stage("Frontend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatefrontendnew.sh"
                                 sh "echo UPDATED FRONTEND ENVs"
                                sh 'echo ====================='
                            }
                        }
                    }
                }
            }
        }
        
        stage("Docker: Build Images"){
            steps{
                script{
                        dir('backend'){
                            dockerbuild("3-tier-secops","${params.BACKEND_DOCKER_TAG}","mashoodkhan")
                        }
                    
                        dir('frontend'){
                            dockerbuild("3-tier-secops","${params.FRONTEND_DOCKER_TAG}","mashoodkhan")
                        }
                }
            }
        }
        
        stage("Docker: Push to DockerHub"){
            steps{
                script{
                    dockerPush("3-tier-secops","${params.BACKEND_DOCKER_TAG}","mashoodk/3-tier-secops") 
                    dockerPush("3-tier-secops","${params.FRONTEND_DOCKER_TAG}","mashoodk/3-tier-secops")
                }
            }
        }

    } // closes stages

    // post {
    //     success {
    //         archiveArtifacts artifacts: '*.xml', followSymlinks: false
    //         build job: "Wanderlust-CD", parameters: [
    //             string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}"),
    //             string(name: 'BACKEND_DOCKER_TAG', value: "${params.BACKEND_DOCKER_TAG}")
    //         ]
    //     }
    // }
}
