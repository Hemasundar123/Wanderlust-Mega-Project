@Library('Shared') _
pipeline {
    agent {label 'Mike'}
    
    environment{
        SONAR_HOME = tool "sonar"
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
        stage("Workspace cleanup"){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        
        stage('Git: Code Checkout') {
            steps {
                script{
                    code_checkout("https://github.com/Hemasundar123/Wanderlust-Mega-Project","main")
                }
            }
        }
        
        stage("Trivy: Filesystem scan"){
            steps{
                script{
                    trivy_scan()
                }
            }
        }

        stage("OWASP: Dependency check"){
            steps{
                script{
                    owasp_dependency()
                }
            }
        }
        
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarqube_analysis("sonar","Mikey","Mikey")
                }
            }
        }
        
        stage("SonarQube: Code Quality Gates"){
            steps{
                script{
                    sonarqube_code_quality()
                }
            }
        }
        
        stage('Exporting environment variables') {
            parallel{
                stage("Backend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatebackendnew.sh"
                            }
                        }
                    }
                }
                
                stage("Frontend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatefrontendnew.sh"
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
                            docker_build("Mikey-backend-beta","${params.BACKEND_DOCKER_TAG}","mikey699")
                        }
                    
                        dir('frontend'){
                            docker_build("Mikey-frontend-beta","${params.FRONTEND_DOCKER_TAG}","mikey699")
                        }
                }
            }
        }
        
        stage("Docker: Push to DockerHub"){
            steps{
                script{
                    docker_push("Mikey-backend-beta","${params.BACKEND_DOCKER_TAG}","mikey699") 
                    docker_push("Mikey-frontend-beta","${params.FRONTEND_DOCKER_TAG}","mikey699")
                }
            }
        }
    }
    post{
        success{
            archiveArtifacts artifacts: '*.xml', followSymlinks: false
            build job: "Mikey-CD", parameters: [
                string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}"),
                string(name: 'BACKEND_DOCKER_TAG', value: "${params.BACKEND_DOCKER_TAG}")
            ]
        }
    }
}
