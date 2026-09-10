def branch = "main"
def remote = "origin"
def directory = "~/wayshub-frontend"
def directory2 = "~/deploy"
def builderserver = "djsn98@20.200.155.211"
def appserver = "djsn98@52.147.124.25"
def cred = "student-ssh"
def repo = "https://github.com/djsn98/wayshub-frontend.git"

pipeline{
    agent any
    triggers {
        githubPush()
    }
    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub-credential')
    }
    stages{
        stage('repo pull'){
            steps{
                sshagent([cred]){
                    sh """ssh -o StrictHostKeyChecking=no ${builderserver} << EOF
		    cd ${directory}
                    git pull ${remote} ${branch}
                    exit
                    EOF"""
                }
            }
        }

        stage('docker build'){
            steps{
                sshagent([cred]){
                    sh """ssh -o StrictHostKeyChecking=no ${builderserver} << EOF
                    cd ${directory}
                    docker build -t djsn98/wayshub-fe:prod .
                    exit
                    EOF"""
                }
            }
        }
	
	stage('docker login'){
            steps{
                sshagent([cred]){
                    sh """ssh -o StrictHostKeyChecking=no ${builderserver} << EOF
                    echo "$DOCKER_CREDENTIALS_PSW" | docker login \
                        -u "$DOCKER_CREDENTIALS_USR" \
                        --password-stdin
                    exit
                    EOF"""
                }
            }
        }

        stage('docker push'){
            steps{
                sshagent([cred]){
                    sh """ssh -o StrictHostKeyChecking=no ${builderserver} << EOF
                    docker push djsn98/wayshub-fe:prod
                    exit
                    EOF"""
                }
            }
        }

        stage('deploy'){
            steps{
                sshagent([cred]){
                    sh """ssh -o StrictHostKeyChecking=no ${appserver} << EOF
		    cd ${directory2}
 	            docker compose -f docker-compose-fe.yaml up -d
                    exit
                    EOF"""
                }
            }
        }
    }
}
