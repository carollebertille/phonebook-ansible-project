@Library('jenkins-shared-library') _ 


pipeline {
    agent none
    
    stages {
        stage('Check bash syntax') {
            agent { docker { image 'koalaman/shellcheck-alpine:latest' } }
            steps {
              script { bashCheck }
            }
        }
        stage('Check yaml syntax') {
            agent { docker { image 'sdesbure/yamllint' } }
            steps {
              script { yamlCheck }
            }
        }
         
         stage('Prepare ansible environment') {
            agent any
            environment {
                VAULTKEY = credentials('vaultkey')
                DEVOPSKEY = credentials('devopskey')
            }
            steps {
                sh 'echo \$VAULTKEY > vault.key'
                sh 'cp \$DEVOPSKEY id_rsa'
                sh  'chmod 600 id_rsa'
            }
         }
         stage('Test and deploy the application') {
           agent any
             stages {
               stage("Install ansible role dependencies") {
                   steps {
                       sh 'ansible-galaxy install  -r roles/requirements.yml'
                   }
               }
                stage("Ping targeted hosts") {
                   steps {
                       sh 'ansible all -m ping -i hosts --private-key id_rsa'
                   }
                }

                 /*stage("Vérify ansible playbook syntax") {
                   steps {
                       sh 'sudo apt install python3-pip -y'
                       sh 'sudo pip install ansible-lint'
                       sh 'ansible-lint -x 306 phonebook.yml'
                       sh 'echo "${GIT_BRANCH}"'
                   }
                 }*/ 
               stage("Build docker images on build host") {
                   when {
                      expression { GIT_BRANCH == 'origin/dev' }
                   }
                   steps {
                       sh 'ansible-playbook  -i hosts --private-key id_rsa --tags "build" --limit build phonebook.yml'
                   }
               }


               stage("Scan docker images on build host") {
                   when {
                      expression { GIT_BRANCH == 'origin/dev' }
                  }
                   steps {
                       sh 'ansible-playbook  -i hosts  --private-key id_rsa --limit build  clair-scan.yml'
                   }

               }
                stage("Push on Artifactory registry") {
                   when {
                      expression { GIT_BRANCH == 'origin/dev' }
                   }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "push" --limit build phonebook.yml'
                   }
               }
               stage("Deploy app in Pre-production Environment") {
                    when {
                       expression { GIT_BRANCH == 'origin/dev' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "preprod" --limit preprod phonebook.yml'
                   }
               }


               stage("Test the functioning of the app in Preprod environment") {
                    when {
                       expression { GIT_BRANCH == 'origin/dev' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "test" --limit preprod phonebook.yml'
                   }
               }

            }

         }
             /*stage("Deploy app in Production Environment") {
                 agent { docker { image 'registry.gitlab.com/robconnolly/docker-ansible:latest' } }
                    when {
                       expression { GIT_BRANCH == 'origin/main' }
                    }
                   steps {

                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "prod" --limit prod phonebook.yml'
                  
                   }
               }
               stage("Ensure application is deployed in production") {
                 agent { docker { image 'registry.gitlab.com/robconnolly/docker-ansible:latest' } }
                  when {
                      expression { GIT_BRANCH == 'origin/master' }
                  }
                  steps {
                      sh 'ansible-playbook  -i hosts --vault-password-file vault.key --tags "prod" check_deploy_app.yml'
                  }
               }*/
            }
         
           
    
    post {
     always {
       script {
         // Use slackNotifier.groovy from shared library and provide current build result as parameter 
         clean
        slacknotifier currentBuild.result
     }
    }

} 
} 
