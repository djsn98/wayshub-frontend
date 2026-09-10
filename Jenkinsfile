def directory = "~/wayshub-frontend"
def builder-server = "djsn98@20.200.155.211"
def app-server = "djsn98@52.147.124.25"
def cred = "student-ssh"
def repo = "https://github.com/djsn98/wayshub-frontend.git"

pipeline{
    agent any
    stages{
        stage('repo clone'){
            steps{
                sshagent([cred]){
                    sh """ssh -o StrictHostKeyChecking=no ${builder-server} << EOF
                    git clone ${repo}
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
                    docker push djsn98/wayshub-fe:prod .
                    exit
                    EOF"""
                }
            }
        }

        stage('docker compose'){
            steps{
                sshagent([cred]){
                    sh """ssh -o StrictHostKeyChecking=no ${app-server} << EOF
                    git clone ${repo}
		    cd ${directory}
	            docker compose up
                    exit
                    EOF"""
                }
            }
        }
    }
}
