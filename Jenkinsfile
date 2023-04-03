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
                mkdir backend ,
                cd backend, 
                git branch: 'main',
                    url: 'https://github.com/Yosef-Adel/SW-BACKEND-Project',
                    credentialsId: 'GG'
                    
            }
        }



        stage('Clean') {
            steps {
               sh 'ls'
            }
        }
        stage('Test') {
            steps {
                sh 'ls'
            }
        }
        
    }
}