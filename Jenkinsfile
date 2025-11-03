#! groovy

pipeline{
    agent any
    environment{
        HOME = "${env.WORKSPACE}"
    }

    stages{
        stage('docker environment'){
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

        stage('build'){
            steps{
                sh """
                docker build -t andrefcoutinho/nif-validator .
                """
            }
        }
    }
}