pipeline{
    agent any
    triggers {
        githubPush()
    }
    environment {
	BUILDER_SERVER = credentials('BUILDER_SERVER')
	APP_SERVER = credentials('APP_SERVER')
	CREDENTIAL = credentials('CREDENTIAL')
        DOCKER_CREDENTIALS = credentials('dockerhub-credential')
    }
    stages{
        stage('repo pull'){
            steps{
                sshagent([CREDENTIAL]){
                    sh """ssh -o StrictHostKeyChecking=no ${BUILDER_SERVER} << EOF
		    cd ${env.WAYSHUB_FE_DIR}
                    git pull ${env.REMOTE} ${env.BRANCH}
                    exit
                    EOF"""
                }
            }
        }

        stage('docker build'){
            steps{
                sshagent([CREDENTIAL]){
                    sh """ssh -o StrictHostKeyChecking=no ${BUILDER_SERVER} << EOF
                    cd ${env.WAYSHUB_FE_DIR}
                    docker build -t djsn98/wayshub-fe:prod .
                    exit
                    EOF"""
                }
            }
        }
	
	stage('docker login'){
            steps{
                sshagent([CREDENTIAL]){
                    sh """ssh -o StrictHostKeyChecking=no ${BUILDER_SERVER} << EOF
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
                sshagent([CREDENTIAL]){
                    sh """ssh -o StrictHostKeyChecking=no ${BUILDER_SERVER} << EOF
                    docker push djsn98/wayshub-fe:prod
                    exit
                    EOF"""
                }
            }
        }

        stage('deploy'){
            steps{
                sshagent([CREDENTIAL]){
                    sh """ssh -o StrictHostKeyChecking=no ${APP_SERVER} << EOF
		    cd ${env.DEPLOY_DIR}
		    docker compose -f docker-compose-fe.yaml down
		    docker image rm djsn98/wayshub-fe:prod
 	            docker compose -f docker-compose-fe.yaml up -d
                    exit
                    EOF"""
                }
            }
        }
    }
    post {
	success {
            withCredentials([
                string(
                    credentialsId: 'discord-webhook',
                    variable: 'DISCORD_WEBHOOK'
                )
            ]) {
                sh '''
                    curl -H "Content-Type: application/json" \
                         -d "{
                           \\"content\\": \\"🚀 Jenkins pipeline berhasil!\\n\\nApplication: wayshub-frontend\\nBuild: #${BUILD_NUMBER}\\nBranch: ${BRANCH_NAME}\\nStatus: SUCCESS\\"
                         }" \
                             "$DISCORD_WEBHOOK"
                    '''
                }
        }
        failure {
            withCredentials([
                string(
                    credentialsId: 'discord-webhook',
                    variable: 'DISCORD_WEBHOOK'
                )
            ]) {
                sh '''
                    curl -H "Content-Type: application/json" \
                         -d "{
                           \\"content\\": \\"❌ Jenkins pipeline gagal!\\n\\nApplication: wayshub-frontend\\nBuild: #${BUILD_NUMBER}\\nBranch: ${BRANCH_NAME}\\nStatus: FAILED\\"
                         }" \
                         "$DISCORD_WEBHOOK"
                '''
            }
        }
    }
}
