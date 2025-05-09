pipeline {
    agent any

    tools {
        maven 'my_maven'
    }

    environment {
        GITNAME = 'ILOHAS-0021'            
        GITEMAIL = 'ilohas0021@naver.com' 
        GITWEBADD = 'https://github.com/ILOHAS-0021/simple_sb.git'
        GITSSHADD = 'git@github.com:ILOHAS-0021/dep.git'
        GITCREDENTIAL = 'github_credentials'
        
        DOCKERHUB = 'ilohas21/spring'
        DOCKERHUBCREDENTIAL = 'docker_credentials'
    }

    stages {
        stage('Checkout Github') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [],
                userRemoteConfigs: [[credentialsId: GITCREDENTIAL, url: GITWEBADD]]])

            }
            post {
                failure {
                    sh "echo clone failed"
                }
                success {
                    sh "echo clone success"
                }
            }
        }
        stage('springboot app build') {
            steps {
                sh "mvn clean package"
            }
            post {
                failure {
                    sh "echo mvn packaging fail"
                }
                success {
                    sh "echo mvn packaging success"
                }
            }
        }
        stage('docker image build') {
            steps {
                sh "docker build -t ${DOCKERHUB}:${currentBuild.number} ."
                sh "docker build -t ${DOCKERHUB}:latest ."
            }
            post {
                failure {
                    sh "echo image build failed"
                }
                success {
                    sh "echo image build success"
                }
            }
        }
        stage('docker image push') {
            steps {
                withDockerRegistry(credentialsId: DOCKERHUBCREDENTIAL, url: '') {
                    sh "docker push ${DOCKERHUB}:${currentBuild.number}"
                    sh "docker push ${DOCKERHUB}:latest"
                }
            }
            post {
                failure {
                    sh "docker image rm -f ${DOCKERHUB}:${currentBuild.number}"
                    sh "docker image rm -f ${DOCKERHUB}:latest"
                    sh "echo push failed"
                }
                success {
                    sh "docker image rm -f ${DOCKERHUB}:${currentBuild.number}"
                    sh "docker image rm -f ${DOCKERHUB}:latest"
                    sh "echo push success"
                }
            }
        }
        stage('EKS manifest file update') {
            steps {
                git credentialsId: GITCREDENTIAL, url: GITSSHADD, branch: 'main'
                sh "git config --global user.email ${GITEMAIL}"
                sh "git config --global user.name ${GITNAME}"
                sh "sed -i 's@${DOCKERHUB}:.*@${DOCKERHUB}:${currentBuild.number}@g' deploy.yml"

                sh "git add ."
                sh "git branch -M main"
                sh "git commit -m 'fixed tag ${currentBuild.number}'"
                sh "git remote remove origin"
                sh "git remote add origin ${GITSSHADD}"
                sh "git push origin main"
            }
            post {
                failure {
                    sh "echo manifest update failed"
                }
                success {
                    sh "echo manifest update success"
                }
            }
        }
    }
}