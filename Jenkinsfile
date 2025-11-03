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
                pip install -r requirements.txt
                """
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

        stage('deploy'){
            steps{
                sh"""
                ssh redhat@63.176.99.227 docker --version
                """
            }
        }
    }
}