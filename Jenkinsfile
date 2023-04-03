pipeline {
    agent {
        docker { image 'yosefadel/aws-node' }
    }
     environment {
      GOCACHE = "${env.WORKSPACE}/.build_cache"
    }
    stages {
        stage('Source Backend') {
            steps {
                sh 'node --version'
                sh 'aws --version'
                git branch: 'main',
                    url: 'https://github.com/Yosef-Adel/SW-BACKEND-Project',
                    credentialsId: 'SW-GitHub'
                    
            }
        }
        stage('Clean') {
            steps {
                dir("${env.WORKSPACE}/Ch04/04_03-docker-agent"){
                    sh 'pwd'
                }
            }
        }
        stage('Test') {
            steps {
                dir("${env.WORKSPACE}/Ch04/04_03-docker-agent"){
                    sh 'npm test'
                }
            }
        }
        
    }
}