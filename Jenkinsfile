def branch = "main"
def remote = "origin"
def directory = "~/wayshub-frontend"
def directory2 = "~/deploy"
def builder-server = "djsn98@20.200.155.211"
def app-server = "djsn98@52.147.124.25"
def cred = "student-ssh"
def repo = "https://github.com/djsn98/wayshub-frontend.git"

pipeline{
    agent any
    stages{
        stage('repo pull'){
            steps{
                sshagent([cred]){
                    sh """ssh -o StrictHostKeyChecking=no ${builder-server} << EOF
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
                    sh """ssh -o StrictHostKeyChecking=no ${builder-server} << EOF
                    cd ${directory}
                    docker build -t djsn98/wayshub-fe:prod .
                    exit
                    EOF"""
                }
            }
        }

        stage('docker push'){
            steps{
                sshagent([cred]){
                    sh """ssh -o StrictHostKeyChecking=no ${builder-server} << EOF
                    docker push djsn98/wayshub-fe:prod
                    exit
                    EOF"""
                }
            }
        }

        stage('deploy'){
            steps{
                sshagent([cred]){
                    sh """ssh -o StrictHostKeyChecking=no ${app-server} << EOF
		    cd ${directory2}
 	            docker compose up -d
                    exit
                    EOF"""
                }
            }
        }
    }
}
