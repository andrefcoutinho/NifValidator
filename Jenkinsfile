#! groovy

pipeline{
    agent any
    environment{
        HOME = "${env.WORKSPACE}"
    }

    stages{
        stage('docker Env'){
            agent {
                docker{
                    image 'python:3.11-slim'
                    reuseNode true
                }
            }
            steps{
                sh"""
                export PIP_DISABLE_PIP_VERSION_CHECK=1
                export PATH="$HOME/.LOCAL/BIN:${PATH}"
                pip install -r requirements.txt
                pip install -r requirements-test.txt
                """
            }
        }
        stage('unit test'){
            agent{
                docker{
                    image 'python:3.11-slim'
                    reuseNode true
                }
            }
            steps{
                sh 'python3 -m pytest --junitxml results.xml tests/'
            }
            post{
                always{
                    archiveArtifacts artifacts: 'results.xml', allowEmptyArchive: true
                    junit 'results.xml'
                }
            }
        }
        stage('Coverage report') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    reuseNode true
                }
            }
            steps {
                sh'''
                python3 -m coverage run --source=. --omit=tests/* -m pytest tests
                python3 -m coverage report -m
                python3 -m coverage html
                '''
            }
            post {
                always {
                    publishHTML(target:[
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'htmlcov',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }
        stage('deliver'){
            steps{
                withCredentials([usernamePassword(
                    credentialsId:'docker-id',
                    passwordVariable:'passwd', 
                    usernameVariable:'username')]){
                        sh """
                        printenv
                        docker build -t ${username}/${JOB_BASE_NAME} .
                        docker login -u ${username} -p ${passwd}
                        docker push ${username}/${JOB_BASE_NAME}
                        """
                    }
            }
        }

        stage('Deploy'){
            steps{
                withCredentials([usernamePassword(
                    credentialsId:'docker-id',
                    passwordVariable:'passwd', 
                    usernameVariable:'username')]){
                sshagent(credentials:['cluster-credentials']){
                    sh"""
                    ssh -o StrictHostKeyChecking=no redhat@63.176.99.227 docker rm -f ${JOB_BASE_NAME}||true
                    ssh -o StrictHostKeyChecking=no redhat@63.176.99.227 docker run -d --name ${JOB_BASE_NAME} -p 8080:9046 ${username}/${JOB_BASE_NAME}
                    """
                }
                }
            }
        }
    }
}