pipeline {
    agent any 
 
     environment {
        GOCACHE = "${env.WORKSPACE}/.build_cache"
        ANSIBLE_PRIVATE_KEY=credentials('ansible-private-key') 
        ENVVAER=credentials('ENVTXTGG') 
      
        }

    stages {
        stage('Source Backend') {
            agent {
                    docker { image 'node' }
                }
            steps {
                
                dir('backend') {

                    git branch: 'main',
                    url: 'https://github.com/Yosef-Adel/SW-BACKEND-Project'
                   
                }
            
                    
            }
        }
        stage('Source Frontend') {
             agent {
                    docker { image 'node' }
                }
            steps {
                dir('frontend') {

                    git branch: 'master',
                    url: 'https://github.com/Yosef-Adel/SW-FRNT-Project'
                    
                }
            
                    
            }
        }

        stage('build-backend') {
             agent {
                    docker { image 'node' }
                }
            steps {
             dir("backend") {
                //  sh 'npm install'
                //  sh 'npm run build'
             }
            }
        }

        stage('build-frontend') {
             agent {
                    docker { image 'node' }
                }
        steps {
            dir("frontend") {
            //  sh 'npm install'
            //  sh 'npm run build'
            }
        }
        }


        stage('test-backend') {
             agent {
                    docker { image 'node' }
                }
            steps {
                dir("backend") {
                // sh 'npm install'
                // sh 'npm  test'
             }
            }
        }

         stage('test-frontend') {
             agent {
                    docker { image 'node' }
                }
            steps {
                dir("frontend") {
                // sh 'npm install'
                // sh 'npm  test'
             }
            }
        }
        stage('scan-backend') {
             agent {
                    docker { image 'node' }
                }
            steps {
                dir("backend") {
                // sh 'npm install'
                // sh 'npm audit fix --audit-level=critical --force'
             }
            }
        }
         stage('scan-frontend') {
             agent {
                    docker { image 'node' }
                }
            steps {
                dir("frontend") {
                // sh 'npm install'
                // sh 'npm audit fix --audit-level=critical --force'
             }
            }
        }

        stage('deploy-infrastructure') {
            agent {
                docker { image 'yosefadel/aws-node' }
            }
            environment {
                AWS_DEFAULT_REGION="us-east-1"
            }
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        aws cloudformation deploy \
                        --template-file files/backend.yml \
                        --tags Project=SW-project \
                        --stack-name "SW-project-backend-${BUILD_ID}" \
                        --parameter-overrides ID="${BUILD_ID}"
                        
                        aws ec2 describe-instances \
                        --filters "Name=tag:Name,Values=backend-${BUILD_ID}" \
                        --query 'Reservations[*].Instances[*].PublicIpAddress' \
                        --output text >> ansible/inventory.txt
                    '''
                    stash name: 'invFile', includes: 'ansible/inventory.txt'
                }
            }
        }
        stage('configure-infrastructure') {
            steps {
                dir('ansible') {
                    unstash 'invFile'
                    sh ''' 
                        ls
                        pwd
                        cd ansible
                        cat inventory.txt 
                        echo "Test Webhook"
                        ansible-playbook -i inventory.txt --private-key=$ANSIBLE_PRIVATE_KEY configure-server.yml
                    '''
                }
            }
        }


      


    //    stage ('deploy-backend') {

    //     steps {
    //         sh ''' 
    //             cd backend 
    //             ls
    //             rm -rf node_modules
    //             cat $ENVVAER >> .env
    //             cd ..
    //             tar -czf artifact.tar.gz  backend
    //             cp artifact.tar.gz ansible/roles/deploy/artifact.tar.gz
    //             ls ansible/roles/deploy/
    //             cd ansible 
    //             echo "Test Webchoock"
    //         '''
    //                 sh 'ansible-playbook -i inventory/mariadb.hosts --private-key=$ANSIBLE_PRIVATE_KEY playbooks/mariadb.yml'

    //         ansiblePlaybook inventory: 'inventory.txt', playbook: 'deploy-backend.yml', sudo: true, vaultCredentialsId: 'ansible-private-key'
    //     }

    //    }
      
    }

}