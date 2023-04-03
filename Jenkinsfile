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
        stage('Source Frontend') {

            steps {
                dir('frontend') {

                    git branch: 'main',
                    url: 'https://github.com/Yosef-Adel/SW-FRNT-Project',
                    credentialsId: 'GG'
                }
            
                    
            }
        }

        stage('build-backend') {
            steps {
             dir("backend") {
                //  sh 'npm install'
                //  sh 'npm run build'
             }
            }
        }

        stage('build-frontend') {
        steps {
            dir("frontend") {
            //  sh 'npm install'
            //  sh 'npm run build'
            }
        }
        }


        stage('test-backend') {
            steps {
                dir("backend") {
                // sh 'npm install'
                // sh 'npm  test'
             }
            }
        }

         stage('test-frontend') {
            steps {
                dir("frontend") {
                // sh 'npm install'
                // sh 'npm  test'
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
         stage('scan-frontend') {
            steps {
                dir("frontend") {
                sh 'npm install'
                sh 'npm audit fix --audit-level=critical --force'
             }
            }
        }

        
    }
}