def destroy_environment(){
    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',AWS_DEFAULT_REGION:'us-east-1')]) {

    sh 'echo "Destroying environment "'
    sh 'aws cloudformation delete-stack --stack-name SW-project-backend-${BUILD_ID}'
    sh ' aws s3 rm s3://sw-project-${BUILD_ID}  --recursive'
    sh 'aws cloudformation delete-stack --stack-name SW-project-frontend-${BUILD_ID}'


    }
}


def fail_alert(stage_name){

    slackSend color: 'danger', iconEmoji: '⛔️', message:  stage_name + 'failed '
    slackSend color: 'danger', iconEmoji: '👩‍🦯', message: 'تعبت يجدعان والله تعبت 💔'

}

def pass_alert(stage_name){
    slackSend color: 'good', iconEmoji: '🥱', message: stage_name +  'Completed '
    slackSend color: 'good', iconEmoji: '👩‍🦯', message: 'اللي بعدوووووووووووووو'
}

pipeline {
    agent any
    environment {
        GOCACHE = "${env.WORKSPACE}/.build_cache"
        CI = 'false'
    }
    
        
    stages {
        stage ('Start Pipeline'){
            steps {
                slackSend color: 'good', iconEmoji: '💪', message: 'الله المستعان نبدأ الشغل ❤️'            
            }
        }

        stage('Source Frontend') {
            steps {
               dir('frontend') {
                    checkout scmGit(branches: [[name: '*/Deplyment']], extensions: [], userRemoteConfigs: [[credentialsId: 'git-cli', url: 'https://github.com/Yosef-Adel/SW-FRNT-Project']])
                    
                }
                stash(name: 'frontend-code', includes: 'frontend/**')
                pass_alert("Source Frontend")
            }
            post {
                failure {
                    fail_alert("Source Frontend")
                }
            }
        }
        
        stage('Source Backend') {
            steps {

                dir('backend') {
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'git-cli', url: 'https://github.com/Yosef-Adel/SW-BACKEND-Project.git']])
                    
                }
                stash(name: 'backend-code', includes: 'backend/**')
               pass_alert("Source Backend")
            }
            post {
                failure {
                    fail_alert("Source Backend")
                }
            }
        }

        stage('Build Frontend') {
            agent {
                docker {
                    image 'node:16.20.0'
                }
            }
            steps {
                
                unstash 'frontend-code'
                dir('frontend') {
                    sh 'ls'
                    sh 'npm install'
                    sh 'npm run build'
                }

                stash(name: 'frontend-build', includes: 'frontend/build/**')
                pass_alert("Build Frontend")
            
            }
            post {
                failure {
                    fail_alert("Build Frontend")
                }
            }
        }
        
        stage('Build Backend') {
            agent {
                docker {
                    image 'node:16.20.0'
                }
            }
            steps {
                
                unstash 'backend-code'
                dir('backend') {
                    sh 'npm install '
                    // sh 'npm build'
                }
                // stash(name: 'backend-build', includes: 'backend/build**')
                pass_alert("Build Backend")
            }
            post {
                failure {
                    fail_alert("Build Backend")
                }
            }
        }


        stage('Test Frontend') {
             agent {
                docker {
                    image 'node:16.20.0'
                }
            }
            steps {
                
                unstash 'frontend-code'
                dir('frontend') {
                    // sh 'npm install'
                    // sh 'npm test'
                   
                }
                pass_alert("Test Frontend ")
                
            }
            post {
                failure {
                    fail_alert("Test Frontend")
                }
            }
            
        }
        
        stage('Test Backend') {
             agent {
                docker {
                    image 'node:16.20.0'
                }
            }
            steps {
                
                unstash 'backend-code'
                dir('backend') {
                    // sh 'npm install'
                    // sh 'npm test '
                   
                }
                pass_alert("Test Backend ")
            }
            post {
                failure {
                    fail_alert("Test Backend")
                }
            }
          
        }

        
        stage('Scan Backend') {
             agent {
                docker {
                    image 'node:16.20.0'
                }
            }
            steps {
                
                unstash 'backend-code'
                dir('backend') {
                    // sh 'npm install '
                    // sh 'npm audit fix --audit-level=critical --force'
                }
                pass_alert("Scan Backend ")
            }
            post {
                failure {
                    fail_alert("Scan Backend")
                }
            }
          
        }
        
        stage('Scan Frontend') {
             agent {
                docker {
                    image 'node:16.20.0'
                }
            }
            steps {
                
                unstash 'frontend-code'
                dir('frontend') {
                    // sh 'npm install '
                    // sh 'npm audit fix --audit-level=critical --force'
                }
                pass_alert("Scan Frontend ")
            }
            post {
                failure {
                    fail_alert("Scan Frontend")
                }
            }
           
        }

        stage('Dockerize Backend') {
            environment {
                DOCKERHUB_CREDENTIALS = credentials('dockerhub')
            }
            steps {
                unstash 'backend-code'
                dir('backend') {
                    sh 'docker build -t yosefadel/sw-project-backend .'
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR  --password-stdin'
                    sh 'docker push yosefadel/sw-project-backend'
                }
                pass_alert("Dockerize Backend ")
            }
            post {
                failure {
                    fail_alert("Dockerize Backend ")
                    destroy_environment()
                }
            }
          
        }
        
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
                    pass_alert("Deploy Infrastructure ")
                  
                }
            }
            post {
                failure {
                    fail_alert("Deploy Infrastructure ")
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
                        ansible-playbook -i inventory.txt --private-key=$ANSIBLE_PRIVATE_KEY configure-server.yml
                    '''
               }
              pass_alert("Configure Infrastructure ")
            }
            post {
                failure {
                    fail_alert("Configure Infrastructure ")
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
                dir('server') {
                    sh 'cat $ENVVAER >> .env'
                }

                sh 'tar -czf artifact.tar.gz server'
                sh 'cp artifact.tar.gz ansible/roles/deploy/artifact.tar.gz'
                
                dir('ansible') {
                   
                    sh 'cat inventory.txt'
                    sh 'ansible-playbook -i inventory.txt --private-key=$ANSIBLE_PRIVATE_KEY deploy-backend.yml'
                }


                pass_alert("Deploy Backend ")
            }
            post {
                failure {
                    fail_alert("Deploy Backend ")
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
            pass_alert("Deploy Frontend  ")     
            }
            post {
                failure {
                    fail_alert("Deploy Frontend  ")
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
        

        stage('Cloudfront Update') {
            agent {
                docker {
                    image 'yosefadel/aws-node'
                }
            }
            environment {
                KVDB_BUCKET= credentials('KVDB_BUCKET')
                AWS_DEFAULT_REGION="us-east-1"
            }
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    
                    def oldWorkflowID = sh(
                    script: "aws cloudformation list-exports \
                        --query \"Exports[?Name==\`WorkflowID\`].Value\" \
                        --no-paginate --output text",
                        returnStdout: true
                        ).trim()
                    sh "curl -k https://kvdb.io/${KVDB_BUCKET}/old_workflow_id -d \"${oldWorkflowID}\""
                    echo "Old Workflow ID: ${oldWorkflowID}"
                    
                    sh ''' 
                    aws cloudformation deploy \
                    --template-file files/cloudfront.yml \
                    --parameter-overrides WorkflowID="${BUILD_ID}" \
                    --stack-name InitialStack
                    '''

                }
            }
            post {
                failure {
                    fail_alert("Deploy Frontend  ")
                    destroy_environment()
                }
            }
        }

        stage('Cleanup') {
            agent {
                docker {
                    image 'yosefadel/aws-node'
                }
            }
            environment {
                KVDB_BUCKET= credentials('KVDB_BUCKET')
                AWS_DEFAULT_REGION="us-east-1"
            }
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    script {
                        def stacks = sh(
                            script: 'aws cloudformation list-stacks --query "StackSummaries[*].StackName" --stack-status-filter CREATE_COMPLETE --no-paginate --output text',
                            returnStdout: true
                        ).trim().split()
                        echo "Stack names: ${stacks}"
                        
                        def oldWorkflowID = sh(
                            script: 'curl --insecure https://kvdb.io/${KVDB_BUCKET}/old_workflow_id',
                            returnStdout: true
                        ).trim()
                        echo "Old Workflow ID: ${oldWorkflowID}"

                        if (stacks.contains(oldWorkflowID)) {
                            sh "aws s3 rm \"s3://sw-project-${oldWorkflowID}\" --recursive"
                            sh "aws cloudformation delete-stack --stack-name \"SW-project-backend-${oldWorkflowID}\""
                            sh "aws cloudformation delete-stack --stack-name \"SW-project-frontend-${oldWorkflowID}\""
                        }
                    }
                }
            }
        }
   


    }
}
