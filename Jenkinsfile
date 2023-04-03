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
                dir("backend"){
                    git branch: 'main',
                    url: 'https://github.com/Yosef-Adel/SW-BACKEND-Project',
                    credentialsId: 'GG'
                }
            
                    
            }
        }



        stage('Clean') {
            steps {
             dir("backend"){{
                 sh 'ls'
             }
            }
        }
        stage('Test') {
            steps {
                dir("backend"){{
                    sh 'pwd'
             }
            }
        }
        
    }
}