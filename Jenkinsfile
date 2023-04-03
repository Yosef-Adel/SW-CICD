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
                dir('backend') {

                    git branch: 'main',
                    url: 'https://github.com/Yosef-Adel/SW-BACKEND-Project',
                    credentialsId: 'GG'
                }
            
                    
            }
        }

        stage('build-backend') {
            steps {
             dir("backend") {
                 sh 'npm install'
                 sh 'npm run build'
             }
            }
        }

        stage('test-backend') {
            steps {
                dir("backend") {
                sh 'npm install'
                sh 'npm run test'
             }
            }
        }
        stage('scan-backend') {
            steps {
                dir("backend") {
                sh 'npm install'
                sh 'npm audit fix --audit-level=critical --force'
             }
            }
        }

        
    }
}