def destroy_environment(){
    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',AWS_DEFAULT_REGION:'us-east-1')]) {

    sh 'echo "Destroying environment "'
    sh 'aws cloudformation delete-stack --stack-name SW-project-backend-${BUILD_ID}'
    sh ' aws s3 rm s3://sw-project-${BUILD_ID} } --recursive'
    sh 'aws cloudformation delete-stack --stack-name SW-project-frontend-${BUILD_ID}'


    }
}






pipeline {
    agent any
    environment {
        GOCACHE = "${env.WORKSPACE}/.build_cache"
        WORKFLOW_ID = "${BUILD_ID}"
        CI = 'false'
    }

        
    stages {
        // stage('Source Frontend') {
        //     steps {
        //        dir('frontend') {
        //             checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'git-cli', url: 'https://github.com/Yosef-Adel/SW-FRNT-Project']])
                    
        //         }
        //         stash(name: 'frontend-code', includes: 'frontend/**')
        //     }
        // }
        
        // stage('Source Backend') {
        //     steps {

        //         dir('backend') {
        //             checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'git-cli', url: 'https://github.com/Yosef-Adel/SW-BACKEND-Project.git']])
                    
        //         }
        //         stash(name: 'backend-code', includes: 'backend/**')
        //     }
        // }
        

        // stage('Build Frontend') {
        //     agent {
        //         docker {
        //             image 'node:16.20.0'
        //         }
        //     }
        //     steps {
        //         sh 'rm -rf frontend '
        //         unstash 'frontend-code'
        //         dir('frontend') {
        //             sh 'ls'
        //             // sh 'npm install'
        //             // sh 'npm run build'
        //         }

        //         stash(name: 'frontend-build', includes: 'frontend/build/**')
        //     }
        // }
        
        // stage('Build Backend') {
        //     agent {
        //         docker {
        //             image 'node:16.20.0'
        //         }
        //     }
        //     steps {
        //         sh 'rm -rf backend '
        //         unstash 'backend-code'
        //         dir('backend') {
        //             sh 'npm install '
        //             sh 'npm build'
        //         }
        //         stash(name: 'backend-build', includes: 'backend/build**')

        //     }
        // }


        // stage('Test Frontend') {
        //      agent {
        //         docker {
        //             image 'node:16.20.0'
        //         }
        //     }
        //     steps {
        //         sh 'rm -rf frontend '
        //         unstash 'frontend-code'
        //         dir('frontend') {
        //             // sh 'npm install '
        //             // sh 'npm test'
                   
        //         }
        //     }
            
        // }
        
        // stage('Test Backend') {
        //      agent {
        //         docker {
        //             image 'node:16.20.0'
        //         }
        //     }
        //     steps {
        //         sh 'rm -rf backend '
        //         unstash 'backend-code'
        //         dir('backend') {
        //             // sh 'npm install '
        //             // sh 'npm test '
                   
        //         }
        //     }
          
        // }

        
        // stage('Scan Backend') {
        //      agent {
        //         docker {
        //             image 'node:16.20.0'
        //         }
        //     }
        //     steps {
        //         sh 'rm -rf backend '
        //         unstash 'backend-code'
        //         dir('backend') {
        //             // sh 'npm install '
        //             // sh 'npm audit fix --audit-level=critical --force'
        //         }
        //     }
          
        // }
        
        // stage('Scan Frontend') {
        //      agent {
        //         docker {
        //             image 'node:16.20.0'
        //         }
        //     }
        //     steps {
        //         sh 'rm -rf frontend '
        //         unstash 'frontend-code'
        //         dir('frontend') {
        //             // sh 'npm install '
        //             // sh 'npm audit fix --audit-level=critical --force'
        //         }
        //     }
           
        // }
        
        stage('Deploy Infrastructure') {
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
                    
                    '''
                    sh ''' 
                        aws cloudformation deploy \
                        --template-file files/frontend.yml \
                        --tags Project=SW-project \
                        --stack-name "SW-project-frontend-${BUILD_ID}" \
                        --parameter-overrides ID="${BUILD_ID}"    
                    
                    '''

                    sh '''
                        echo "[web]" >> ansible/inventory.txt

                        aws ec2 describe-instances \
                        --filters "Name=tag:Name,Values=backend-${BUILD_ID}" \
                        --query 'Reservations[*].Instances[*].PublicIpAddress' \
                        --output text >> ansible/inventory.txt
                    '''
                    stash name: 'invFile', includes: 'ansible/inventory.txt'
                }
            }
            post {
                failure {
                    
                    destroy_environment()
                }
            }

           
        }
        
        stage('Configure Infrastructure') {
             agent {
                docker {
                    image 'yosefadel/aws-node'
                }
            }
            environment {
                
                ANSIBLE_PRIVATE_KEY=credentials('ansible-private-key')
            }
            steps {
                unstash 'invFile'
                dir('ansible') {
                    sh ''' 
                        cat inventory.txt 
                        cat $ANSIBLE_PRIVATE_KEY
                        echo "Test Webhook"
                        ansible-playbook -i inventory.txt --private-key=$ANSIBLE_PRIVATE_KEY configure-server.yml
                    '''
               }
            }
            post {
                failure {
                    
                    destroy_environment()
                }
            }
          
        }
        
        
        stage('Deploy Backend') {
             agent {
                docker {
                    image 'yosefadel/aws-node'
                }
            }
            environment {
                
                ANSIBLE_PRIVATE_KEY=credentials('ansible-private-key')
                ENVVAER=credentials('ENVTXTGG') 
            }
            steps {
                unstash 'invFile'
                dir('backend') {
                  
                    sh 'rm -rf node_modules .git'
                    sh 'cat $ENVVAER >> .env'
                }

                sh 'tar -czf artifact.tar.gz backend'
                sh 'cp artifact.tar.gz ansible/roles/deploy/artifact.tar.gz'
                
                dir('ansible') {
                   
                    sh 'cat inventory.txt'
                    sh 'ansible-playbook -i inventory.txt --private-key=$ANSIBLE_PRIVATE_KEY deploy-backend.yml'
                }
            }
            post {
                failure {
                    
                    destroy_environment()
                }
            }
          
        }


        
        stage('Deploy Frontend') {
             agent {
                docker {
                    image 'yosefadel/aws-node'
                }
            }
            environment {
                AWS_DEFAULT_REGION="us-east-1"
                
            }
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    unstash 'frontend-build'
                    sh 'ls'
                    dir('frontend') {
                        sh 'ls'
                        sh 'echo "API_URL=http://ec2-3-219-197-102.compute-1.amazonaws.com/" >> .env'
                        sh 'tar -czvf artifact-"${BUILD_ID}".tar.gz build'
                        sh 'aws s3 cp build s3://sw-project-${BUILD_ID} --recursive'
                        
                    }
                }
            }
            post {
                failure {
                    
                    destroy_environment()
                }
            }
        }
        


        // stage('Smoke Test') {
        //     steps {
        //         // add commands to run smoke test
        //     }
        //     dependencies {
        //         stage('Deploy Backend')
        //         stage('Deploy Frontend')
        //     }
        // }
        

        // stage('Cloudfront Update') {
        //     steps {
        //         // add commands to update cloudfront
        //     }
        //     dependencies {
        //         stage('Smoke Test')
        //     }
        // }
        

        // stage('Cleanup') {
        //     steps {
        //         // add commands to clean up
        //     }
        //     dependencies {
        //         stage('Cloudfront Update')
        //     }
        // }


    }
}
